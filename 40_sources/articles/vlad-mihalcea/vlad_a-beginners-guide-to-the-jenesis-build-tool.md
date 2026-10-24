---
tags:
  - java
  - build-tools
  - jenesis
  - hibernate
  - hypersistence-utils
  - java-25
feature:
type: article
author: Vlad Mihalcea
source: https://vladmihalcea.com/jenesis-build-tool/
date: 2026-10-24
---

# A beginner's guide to the Jenesis build tool

## Sunto

Jenesis è un nuovo strumento di build Java creato da Rafael Winterhalter, Java Champion e autore di Byte Buddy. A differenza di Maven o Gradle, il motore di Jenesis è puro codice Java incorporato nel progetto nella cartella `build/jenesis`, senza necessità di installazione esterna. Jenesis richiede Java 25 o versioni successive e può leggere i file `pom.xml` esistenti, permettendo di usare Maven e Jenesis in parallelo sullo stesso progetto senza migrazioni.

L'articolo documenta l'integrazione di Jenesis nel progetto Hypersistence Utils, una libreria Hibernate con tre moduli e quasi 1.000 test di integrazione eseguibili su PostgreSQL, MySQL, Oracle e SQL Server (localmente o via Testcontainers). La configurazione avviene tramite file properties, poiché Jenesis ignora la sezione `<build><plugins>` di Maven: occorre replicare manualmente il comportamento dei plugin (staging dei test jar, flag JVM per i test, variabili d'ambiente, manifest `Automatic-Module-Name`).

Durante l'integrazione sono emerse alcune problematiche da risolvere: l'esecuzione parallela dei test su più moduli causava conflitti sui database condivisi (risolto con `jenesis.process.concurrency=1`); il compilatore Java ometteva il flag `-g` (tabelle variabili locali) necessario per il bytecode enhancement di Hibernate (risolto con un file `process-javac.properties`); la risoluzione delle dipendenze differisce da Maven per tre versioni che sono state pinnate manualmente nel POM tramite il comando `dependencies pin`.

Un punto di forza rilevante è la riproducibilità della build: un secondo build da zero produce output identici, e i jar includono automaticamente un SBOM CycloneDX. I tempi di build sono comparabili a Maven (741 s vs 751-794 s per un cold build), ma i rebuild incrementali sono molto più veloci — circa 1,4 s per un rebuild senza modifiche grazie allo skip automatico degli step invariati.

La CI su GitHub Actions richiede solo il JDK Temurin 25 senza installazioni aggiuntive. I test incrementali (`jenesis.test.incremental`) non sono ancora affidabili per questo progetto perché non rilevano le classi referenziate solo tramite annotazioni `@Type` di Hibernate, il cui nome classe è presente solo nel constant pool del bytecode. L'autore raccomanda di eseguire un tool alla volta quando i moduli di test condividono un database, e di verificare attentamente l'impatto del comando `pin` su Maven prima di usarlo.

## Immagini

![Rafael Winterhalter a JavaZone 2026](https://vladmihalcea.com/wp-content/uploads/2026/10/Rafael_Winterhalter_JavaZone_2026.jpg)

## Codice

Avviare la build con Jenesis (non richiede installazione esterna, solo JDK 25+):

```bash
java build/jenesis/Make.java
```

Installazione di Jenesis con verifica della versione:

```bash
curl -fsSL https://get.jenesis.build | JENESIS_VERSION=0.17.0 bash
```

Configurazione principale in `jenesis.properties` — staging dei test jar e concorrenza seriale per evitare conflitti tra moduli sullo stesso database:

```properties
jenesis.stage.tests=true
jenesis.process.concurrency=1
```

Aggiunta del flag `-g` al compilatore per preservare le tabelle delle variabili locali (necessario per Hibernate bytecode enhancement):

```properties
-g=
```

Referenza tramite annotazione `@Type` che inganna il rilevamento incrementale di Jenesis — il nome della classe è nel constant pool ma non in un import esplicito:

```java
@Type(PostgreSQLHStoreType.class)
@Column(columnDefinition = "hstore")
private Map<String, String> properties = new HashMap<>();
```

Configurazione CI su GitHub Actions con JDK Temurin 25:

```yaml
- uses: actions/setup-java@v5
  with:
    distribution: temurin
    java-version: '25'

- name: Build and test hypersistence-utils-hibernate-${{ matrix.hibernate }}
  run: >
    java
    -Djenesis.make.profiles=test
    -Djenesis.print.tests=true
    -Djenesis.executor.aggregate=true
    build/jenesis/Make.java
    build/maven/compose/module/build-testmodule-hypersistence-utils-hibernate-${{ matrix.hibernate }}
```

Pinning delle versioni delle dipendenze che differiscono tra Jenesis e Maven:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>jakarta.activation</groupId>
            <artifactId>jakarta.activation-api</artifactId>
            <version>2.1.3</version>
        </dependency>
        <dependency>
            <groupId>net.bytebuddy</groupId>
            <artifactId>byte-buddy</artifactId>
            <version>1.18.8</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

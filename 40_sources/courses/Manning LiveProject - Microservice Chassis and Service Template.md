---
titolo: Learn how to create a service template and microservice chassis
piattaforma: Manning LiveProject
istruttore: Chris Richardson
data: 2026-05-03
durata_totale: hands-on project
trascrizione_coperta: 4 moduli, 16 milestone analizzate
lingua_originale: en
iterazione: 2
tags:
  - microservice-chassis
  - service-template
  - kotlin
  - spring-boot
  - observability
  - security
  - docker
  - hexagonal-architecture
feature: 
type: project
author: Chris Richardson
source: 
---

# Learn how to create a service template and microservice chassis

## Chi è l'istruttore

Chris Richardson è l'autore di *Microservices Patterns* (Manning, 2018), uno dei testi di riferimento sull'architettura a microservizi. È il creatore del pattern Saga e del framework Eventuate. Gestisce il sito microservices.io ed è un consulente esperto di architetture distribuite. Il progetto riflette il suo approccio sistematico: architettura esagonale, test a più livelli (unit, integration, component), e la separazione netta tra "ciò che cambia" (il dominio applicativo) e "ciò che rimane costante" (il chassis).

## Struttura del progetto

Il LiveProject è diviso in quattro moduli progressivi, ciascuno con milestone numbered. I moduli si costruiscono l'uno sull'altro: il codice di ogni milestone successiva include tutto quello della precedente.

| Modulo | Titolo | Milestone | Cosa costruisce |
|--------|--------|-----------|-----------------|
| 01 | Observability | 2, 3, 4 | Health check, Prometheus metrics, distributed tracing |
| 02 | Security | 1, 2, 3, 4 | OAuth2/JWT con Keycloak, autorizzazione role-based e instance-based |
| 03 | Docker packaging | 1, 2, 3, 4 | Dockerfile layered, docker-compose con il servizio, component test, CI/CD |
| 04 | Microservice Chassis | 1, 2, 3, 4, 5 | Estrazione del chassis come libreria riusabile, Gradle plugins, BOM, pubblicazione |

**Tecnologie usate:** Kotlin, Spring Boot 2.x, Gradle multi-module (Kotlin DSL + Groovy DSL), MySQL, Testcontainers, Keycloak, Prometheus, Zipkin, Docker, GitHub Actions.

## Architettura del service template

### Struttura esagonale (Hexagonal / Ports & Adapters)

Il pattern architetturale di base è l'architettura esagonale. Il dominio è al centro e non conosce nulla del layer web, della persistenza o della sicurezza. Questi ultimi sono "adapter" che si connettono al dominio tramite interfacce ("port").

```
┌─────────────────────────────────────────────────────┐
│                 service-template-main                │
│  (entry point: aggrega tutti i subproject)          │
│                                                     │
│  ┌───────────┐  ┌──────────────┐  ┌─────────────┐  │
│  │ web       │  │ web-security │  │ metrics     │  │
│  │ (adapter) │  │ (adapter)    │  │ (observer)  │  │
│  └─────┬─────┘  └──────┬───────┘  └──────┬──────┘  │
│        │               │                 │          │
│  ┌─────▼───────────────▼─────────────────▼──────┐  │
│  │              domain (port)                    │  │
│  │  Account, AccountService, AccountServiceObserver│  │
│  │  AuthenticatedUserSupplier, AccountRepository │  │
│  └─────────────────────┬─────────────────────────┘  │
│                        │                            │
│  ┌─────────────────────▼─────────────────────────┐  │
│  │     persistence (adapter JPA/MySQL)           │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Subproject Gradle (milestone finale, modulo 01)

```
live-projects-service-template/
├── service-template-util          # Jackson ObjectMapper con KotlinModule
├── service-template-domain        # Account entity, AccountService, interfacce port
├── service-template-config        # application.properties
├── service-template-persistence   # DomainPersistenceConfiguration (JPA)
├── service-template-web           # AccountController (REST)
├── service-template-health-check  # Spring Actuator health endpoint
├── service-template-metrics       # Micrometer/Prometheus + MicrometerBasedAccountServiceObserver
├── service-template-distributed-tracing  # Spring Cloud Sleuth + Zipkin
├── service-template-test-util     # Eventually helper per test asincroni
├── service-template-test-data     # TestData factory
├── service-template-test-containers  # KeyCloakContainer, MySqlContainer, ZipkinContainer
└── service-template-main          # @SpringBootApplication, aggrega tutto
```

---

## Modulo 1 — Observability

### Punto di partenza (milestone 2 — già completata)

La milestone 2 è il punto di partenza: il progetto ha già la struttura multi-module, il dominio `Account` e il controller REST. Il modulo `service-template-health-check` è presente ma senza sorgenti Kotlin propri — si affida all'autoconfigure di Spring Boot Actuator. Il modulo `service-template-distributed-tracing` ha solo test di integrazione, le dipendenze sono in `build.gradle.kts`.

```kotlin
// build.gradle.kts di service-template-distributed-tracing
dependencies {
    // nessuna dipendenza main: Spring Cloud Sleuth si autoconfigura
    testImplementation(project(":service-template-domain"))
    testImplementation(project(":service-template-web"))
    testImplementation(project(":service-template-util"))
    testImplementation(project(":service-template-test-util"))
    testImplementation(project(":service-template-test-containers"))
}
```

**Spiegazione:** Spring Cloud Sleuth intercetta automaticamente le chiamate HTTP e aggiunge header `X-B3-TraceId` e `X-B3-SpanId`. Non serve scrivere codice: basta la dipendenza `spring-cloud-starter-sleuth` + `spring-cloud-sleuth-zipkin` nel classpath.

Il dominio è già completo con il pattern "sealed class per i risultati":

```kotlin
// Domain.kt — Account entity con pattern factory method
@Entity
class Account(var balance: Long, var owner: String,
              @Id @GeneratedValue var id: Long? = null) {
    constructor() : this(0, "")

    companion object {
        fun createAccount(balance: Long, owner: String): AccountCommandResult {
            if (balance <= 0) return AmountNotGreaterThanZero(balance)
            else return AccountCreationSuccessful(Account(balance, owner))
        }
    }

    fun debit(amount: Long): AccountCommandResult { /* ... */ }
    fun credit(amount: Long): AccountCommandResult { /* ... */ }
}

sealed class AccountCommandResult {
    data class AccountCreationSuccessful(val account: Account) : AccountCommandResult()
    object Success : AccountCommandResult()
    data class AmountNotGreaterThanZero(val amount: Long) : AccountCommandResult()
    data class BalanceExceeded(val amount: Long, val balance: Long) : AccountCommandResult()
    object Unauthorized : AccountCommandResult()
}
```

**Perché le sealed class?** Permettono di usare `when` exhaustive senza `else`, garantendo che tutti i casi siano gestiti a compile time. È il pattern "Result type" di Kotlin applicato al dominio.

Il service usa il pattern **Observer** per notificare metriche e altri "osservatori" senza dipendere da Micrometer direttamente:

```kotlin
// AccountServiceImpl.kt — interfaccia Observer nel dominio
interface AccountServiceObserver {
    fun noteAccountCreated()
    fun noteSuccessfulDebit()
    fun noteFailedDebit()
    fun noteFailedCredit()
    fun noteSuccessfulCredit()
    fun noteUnauthorizedAccountAccess()
}
```

L'implementazione di `AccountServiceObserver` è `null`-safe (iniettata come nullable): il dominio non dipende da Micrometer.

```kotlin
@Service
@Transactional
class AccountServiceImpl @Autowired constructor(
    val accountRepository: AccountRepository,
    val authenticatedUserSupplier: AuthenticatedUserSupplier = AuthenticatedUserSupplier.EMPTY_SUPPLIER,
    val accountServiceObserver: AccountServiceObserver?  // nullable!
) : AccountService { ... }
```

L'`AuthenticatedUserSupplier` ha un `EMPTY_SUPPLIER` come default — pattern Null Object per evitare null checks quando la sicurezza non è ancora configurata.

### Milestone 3 — Custom Prometheus metrics

Aggiunge il codice sorgente nel modulo `service-template-metrics`:

```kotlin
// MetricsConfiguration.kt — tag comune per tutte le metriche
@Configuration
class MetricsConfiguration {

    @Bean
    fun meterRegistryCustomizer(
        @Value("\${spring.application.name}") serviceName: String
    ): MeterRegistryCustomizer<MeterRegistry> {
        return MeterRegistryCustomizer { registry ->
            registry.config().commonTags("service", serviceName)
        }
    }
}
```

**Perché il tag `service`?** In Prometheus/Grafana si vuole poter filtrare le metriche per nome del servizio. Usando `MeterRegistryCustomizer`, il tag viene aggiunto automaticamente a tutte le metriche senza doverlo specificare manualmente in ogni counter/gauge.

```kotlin
// MicrometerBasedAccountServiceObserver.kt — implementa il pattern Observer con Micrometer
@Component
class MicrometerBasedAccountServiceObserver(meterRegistry: MeterRegistry) : AccountServiceObserver {

    val createCounter = meterRegistry.counter("account.created")
    val successfulDebitCounter = meterRegistry.counter("account.debit.success")
    val failedDebitCounter = meterRegistry.counter("account.debit.failure")
    val successfulCreditCounter = meterRegistry.counter("account.credit.success")
    val failedCreditCounter = meterRegistry.counter("account.credit.failure")
    val unauthorizedCounter = meterRegistry.counter("account.unauthorized")

    override fun noteAccountCreated() = createCounter.increment()
    override fun noteSuccessfulDebit() { successfulDebitCounter.increment() }
    override fun noteFailedDebit() { failedDebitCounter.increment() }
    override fun noteFailedCredit() { failedCreditCounter.increment() }
    override fun noteSuccessfulCredit() { successfulCreditCounter.increment() }
    override fun noteUnauthorizedAccountAccess() { unauthorizedCounter.increment() }
}
```

**Il pattern chiave:** Il dominio definisce `AccountServiceObserver` come interfaccia pura, senza import di Micrometer. `MicrometerBasedAccountServiceObserver` è nel subproject `service-template-metrics` e implementa l'interfaccia. Così il dominio rimane testabile senza Micrometer nel classpath.

### Milestone 4 — Configurazione distributed tracing

Aggiunge in `application.properties` la configurazione di Sleuth + Zipkin:

```properties
spring.sleuth.enabled=true
spring.sleuth.sampler.probability=1   # campiona il 100% delle richieste (dev only)
spring.zipkin.baseUrl=http://localhost:9411/
```

**Nota:** In produzione `sampler.probability` dovrebbe essere tra 0.01 e 0.1 (1-10%) per ridurre l'overhead. Il valore `1` è solo per sviluppo/debug.

Il `service-template-distributed-tracing/build.gradle.kts` dipende da Spring Cloud Sleuth + Zipkin:

```kotlin
dependencies {
    implementation("org.springframework.cloud:spring-cloud-starter-sleuth")
    implementation("org.springframework.cloud:spring-cloud-sleuth-zipkin")
}
```

L'`application.properties` espone anche il Prometheus endpoint:

```properties
management.endpoints.web.exposure.include=info,env,health,prometheus
```

---

## Modulo 2 — Security

### Milestone 1 — Infrastruttura di test con Keycloak

Prima ancora di scrivere la `SecurityConfig`, viene predisposta l'infrastruttura di test con Keycloak tramite Testcontainers.

```kotlin
// KeyCloakContainer.kt — Testcontainer singleton
object KeyCloakContainer : DefaultPropertyProvidingContainer() {

    override val container = GenericContainer<Nothing>(
        ImageFromDockerfile()
            .withDockerfile(FileSystems.getDefault().getPath("../keycloak/Dockerfile"))
    ).apply {
        withReuse(true)  // riusa il container tra i test (ottimizzazione importante!)
        withEnv("KEYCLOAK_ADMIN", "admin")
        withEnv("KEYCLOAK_ADMIN_PASSWORD", "admin")
        withEnv("DB_VENDOR", "h2")
        withExposedPorts(8091)
        ContainerNetwork.withNetwork(this)
        withNetworkAliases("keycloak")
        waitingFor(Wait.forHttp("/admin/"))
    }

    override fun consumeProperties(registry: PropertyConsumer) {
        val port = getPort()
        registry.add("spring.security.oauth2.resourceserver.jwt.issuer-uri",
                     "http://localhost:$port/realms/${getRealm()}")
        registry.add("spring.security.oauth2.resourceserver.jwt.jwk-set-uri",
                     "http://localhost:$port/realms/${getRealm()}/protocol/openid-connect/certs")
        registry.add("keycloak.auth-server-url", getKeyCloakUrl())
    }
}
```

**`withReuse(true)`:** Evita di ricreare il container Keycloak ad ogni esecuzione di test. Il container viene identificato con un hash e riusato se esiste. Riduce i tempi di test da minuti a secondi.

**`DefaultPropertyProvidingContainer` / `PropertyProvidingContainer`:** Interfaccia custom del progetto che incapsula sia l'avvio del container che la propagazione delle sue proprietà (hostname, porta, URL) alla configurazione Spring, usando il pattern `DynamicPropertyRegistry` di Testcontainers.

Il `JwtProvider` è un helper di test che interagisce con Keycloak Admin API per creare utenti e ottenere JWT:

```kotlin
// JwtProvider.kt — helper test per ottenere JWT da Keycloak
@Component
@ConditionalOnProperty("keycloak.auth-server-url")
class JwtProvider(
    @Value("\${keycloak.auth-server-url}") val keycloakUrl: String,
    @Value("\${keycloak.realm}") val realm: String,
    @Value("\${keycloak.resource}") val clientId: String,
    val httpProxy: URI? = null
) {
    fun getJwt(userName: String, password: String, role: String): String {
        ensureUserExists(userName, password, role)  // crea l'utente se non esiste
        return fetchJwt(clientId, userName, password)  // ottieniil JWT
    }

    private fun fetchJwt(clientId: String, userName: String, password: String): String {
        return RestAssured.given()
            .param("client_id", clientId)
            .param("username", userName)
            .param("password", password)
            .param("grant_type", "password")
            .post("${keycloakUrl}/realms/${realm}/protocol/openid-connect/token")
            .then().statusCode(200)
            .extract().path("access_token")
    }
}
```

La `application.properties` ora include il riferimento al Keycloak locale:

```properties
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=http://localhost:8091/realms/service-template/protocol/openid-connect/certs
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8091/realms/service-template
```

### Milestone 2 — SecurityConfig: autenticazione JWT

La prima `SecurityConfig` autentica le richieste ma non verifica i ruoli:

```kotlin
// SecurityConfig.kt — milestone 2: autenticazione senza autorizzazione RBAC
@Configuration
@EnableWebSecurity
class SecurityConfig : WebSecurityConfigurerAdapter() {

    override fun configure(http: HttpSecurity) {
        http
            .authorizeRequests()
            .antMatchers("/actuator/**").permitAll()   // metriche e health: pubblici
            .antMatchers("/swagger**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
            .antMatchers("/**").authenticated()        // tutto il resto: autenticato
            .and()
            .oauth2ResourceServer().jwt()              // valida il JWT
    }
}
```

**Perché `/actuator/**` è pubblico?** Prometheus deve poter fare scraping senza autenticazione. In produzione si potrebbe proteggere con IP filtering a livello di infrastructure.

### Milestone 3 — Role-Based Access Control (RBAC)

Aggiunge il `JwtAuthenticationConverter` per estrarre i ruoli Keycloak e imporre `hasRole`:

```kotlin
// SecurityConfig.kt — milestone 3: RBAC con ruoli Keycloak
@Configuration
@EnableWebSecurity
class SecurityConfig : WebSecurityConfigurerAdapter() {

    fun jwtAuthenticationConverter(): JwtAuthenticationConverter {
        val jwtConverter = JwtAuthenticationConverter()
        jwtConverter.setJwtGrantedAuthoritiesConverter { jwt ->
            // Keycloak mette i ruoli realm sotto jwt.claims["realm_access"]["roles"]
            val realmAccess = jwt.claims["realm_access"] as Map<String, List<String>>?
            val roles = realmAccess?.get("roles")
            (roles ?: listOf()).map { SimpleGrantedAuthority("ROLE_$it") }
        }
        return jwtConverter
    }

    override fun configure(http: HttpSecurity) {
        http
            .authorizeRequests()
            .antMatchers("/actuator/**").permitAll()
            .antMatchers("/swagger**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
            .antMatchers("/**").hasRole("service-template-user")  // ruolo richiesto
            .and()
            .oauth2ResourceServer().jwt { it.jwtAuthenticationConverter(jwtAuthenticationConverter()) }
    }
}
```

**Il mapping dei ruoli Keycloak:** Spring Security si aspetta autorità nella forma `ROLE_xxx`. Keycloak nel JWT mette i ruoli in `realm_access.roles` (diversamente da come farebbe Spring Security OAuth nativo). Il converter personalizzato fa il mapping: ogni ruolo `service-template-user` diventa l'autorità `ROLE_service-template-user`. L'`hasRole("service-template-user")` internamente cerca `ROLE_service-template-user`.

### Milestone 4 — DefaultAuthenticatedUserSupplier

Aggiunge l'implementazione dell'interfaccia `AuthenticatedUserSupplier` che legge l'utente autenticato dal `SecurityContextHolder`:

```kotlin
// SecurityConfig.kt — milestone 4: aggiunto DefaultAuthenticatedUserSupplier
@Component
class DefaultAuthenticatedUserSupplier : AuthenticatedUserSupplier {

    override fun get(): AuthenticatedUser {
        val authentication = SecurityContextHolder.getContext().authentication
        return AuthenticatedUser(
            id = authentication.name,
            roles = authentication.authorities.map(GrantedAuthority::getAuthority).toSet()
        )
    }
}
```

**Perché questa separazione?** Il dominio usa `AuthenticatedUserSupplier` come interfaccia per ottenere l'utente corrente — è una "port" del dominio. L'implementazione `DefaultAuthenticatedUserSupplier` sta nel modulo `service-template-web-security` perché conosce Spring Security. In test si può sostituire con uno stub che ritorna un utente fisso. Questo rispetta la separazione delle responsabilità dell'architettura esagonale.

L'`AccountServiceImpl` usa il supplier per la **instance-based security**: un utente può accedere solo ai propri account.

```kotlin
private fun withAuthorizedAccess(id: Long, function: (Account) -> AccountServiceCommandResult)
        : Optional<AccountServiceCommandResult> {
    return accountRepository.findById(id).map { account ->
        if (account.owner != currentUserId())
            AccountServiceCommandResult.Unauthorized  // ownership check nel dominio
        else
            function(account)
    }
}
```

---

## Modulo 3 — Docker packaging

### Milestone 1 — Dockerfile multi-stage con Spring Boot layered jar

```dockerfile
# Dockerfile — multi-stage build con Spring Boot layertools
FROM amazoncorretto:11.0.14-al2 as builder
ARG JAR_FILE=build/libs/service-template-main-0.0.1-SNAPSHOT.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM amazoncorretto:11.0.14-al2

# Non-root user per sicurezza
RUN yum install -y shadow-utils && yum clean all && rm -rf /var/cache/yum
RUN groupadd springboot && useradd --system -g springboot springboot
USER springboot

HEALTHCHECK --start-period=30s --interval=5s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Layer ottimizzate per la cache Docker: dalle meno volatili alle più volatili
COPY --from=builder dependencies/ ./
RUN true
COPY --from=builder snapshot-dependencies/ ./
RUN true
COPY --from=builder spring-boot-loader/ ./
RUN true
COPY --from=builder application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

**Spring Boot layered jar:** `java -Djarmode=layertools -jar application.jar extract` scompone il fat JAR in quattro layer ordinati per frequenza di cambiamento:
- `dependencies/` — dipendenze release (cambiano raramente → layer stabile)
- `snapshot-dependencies/` — dipendenze SNAPSHOT (cambiano più spesso)
- `spring-boot-loader/` — il classloader di Spring Boot (quasi mai cambia)
- `application/` — il codice applicativo (cambia ad ogni build)

Docker riusa i layer precedenti (cache hit) se non sono cambiati. Senza questo approccio, ogni build ricaricherebbe decine di MB di dipendenze Maven.

**`RUN true` tra i COPY:** Workaround per un bug di Moby (il runtime Docker) che causa errori intermittenti con file copiati da stage precedenti su alcune versioni.

Il `build.gradle.kts` del modulo `main` abilita i layered jar:

```kotlin
tasks.getByName<BootJar>("bootJar") {
    layered {
        isEnabled = true
    }
}
```

### Milestone 2 — Aggiunta del servizio al docker-compose

Aggiunge il servizio all'`docker-compose.yml` per poter eseguire il servizio insieme all'infrastruttura:

```yaml
service-template:
  build: ./service-template-main
  image: live-projects-service-template
  ports:
    - 8080:8080
  depends_on:
    - keycloak
    - mysql
    - zipkin
  environment:
    # Override delle properties via env vars (convenzione Spring: . -> _ e tutto uppercase)
    - SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK_SET_URI=http://keycloak:8091/realms/service-template/protocol/openid-connect/certs
    - SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER_URI=http://keycloak:8091/realms/service-template
    - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/service_template
    - SPRING_ZIPKIN_BASE_URL=http://zipkin:9411/
```

**Nota importante:** Nel docker-compose gli URL usano i nomi dei servizi Docker (`keycloak`, `mysql`, `zipkin`) al posto di `localhost`. Spring Boot converte automaticamente le variabili d'ambiente in properties: `SPRING_DATASOURCE_URL` diventa `spring.datasource.url`.

### Milestone 3 — Component test con DockerComposeContainer

Un *component test* verifica il sistema completo tramite docker-compose, dall'esterno (black-box):

```kotlin
// ServiceTemplateComponentTest.kt — test end-to-end con docker-compose
@Testcontainers
class ServiceTemplateComponentTest {

    companion object {
        @JvmStatic
        @Container
        val dockerComposeContainer = DockerComposeContainer<Nothing>(File("../docker-compose.yml")).apply {
            withExposedService("service-template_1", 8080, Wait.forHealthcheck())
            withExposedService("keycloak_1", 8091, Wait.forHttp("/"))
            withExposedService("keycloak-test-proxy_1", 80, Wait.forHttp("/"))
            withBuild(true)  // rebuild dell'immagine Docker prima del test
        }
    }

    @Test
    fun shouldRetrieveAccounts() {
        given()
            .header("Authorization", "Bearer ${fetchJwt()}")
            .get(serviceContainer.hostUrl("/accounts"))
            .then().statusCode(200)
    }

    @Test
    fun accountsRequireAuthentication() {
        get(serviceContainer.hostUrl("/accounts")).then().statusCode(401)
    }

    @Test
    fun shouldHavePrometheusMetrics() {
        get(serviceContainer.hostUrl("/actuator/prometheus")).then().statusCode(200)
    }

    @Test
    fun shouldHaveHealthCheckEndpoint() {
        get(serviceContainer.hostUrl("/actuator/health")).then().statusCode(200)
    }
}
```

**`Wait.forHealthcheck()`:** Testcontainers aspetta che il HEALTHCHECK del Dockerfile ritorni OK prima di eseguire i test. Questo è possibile grazie al `HEALTHCHECK` definito nel Dockerfile.

**`withBuild(true)`:** Fa il build dell'immagine Docker prima di avviare il compose. In CI/CD questo garantisce di testare l'immagine appena buildata.

### Milestone 4 — CI/CD con GitHub Actions

```yaml
# .github/workflows/build.yml
- name: Build
  run: |
    cp service-template-test-containers/src/main/resources/dot.testcontainers.properties ~/.testcontainers.properties
    ./gradlew compileAll
    export DOCKER_HOST_IP=$(hostname -I | sed -e 's/ .*//g')
    echo DOCKER_HOST_IP=$DOCKER_HOST_IP
    ./gradlew build
    .github/workflows/verify-todos.py

- name: Publish image
  run: |
    echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u cer --password-stdin
    docker tag live-projects-service-template ghcr.io/$GITHUB_REPOSITORY
    docker push ghcr.io/$GITHUB_REPOSITORY
```

**`DOCKER_HOST_IP`:** Alcune versioni di Testcontainers hanno bisogno di sapere l'IP del Docker host per fare port-mapping. Su Linux si ottiene con `hostname -I`.

**`dot.testcontainers.properties`:** File copiato nella home dell'utente CI. Contiene tipicamente `testcontainers.reuse.enable=true` per riusare i container tra i test.

**`verify-todos.py`:** Script custom che verifica che i TODO nei sorgenti siano stati rimossi prima del merge.

---

## Modulo 4 — Microservice Chassis

Questo è il modulo centrale del progetto. L'obiettivo è estrarre tutto ciò che è "meccanismo di framework" (sicurezza, metriche, tracing, health check, persistence) in una libreria riusabile chiamata **service chassis**. I servizi applicativi usano il chassis come dipendenza Maven/Gradle e si concentrano solo sul dominio.

### Milestone 1 — Creazione del chassis (struttura iniziale)

Il chassis viene creato come nuovo progetto Gradle separato con package `net.chrisrichardson.liveprojects.servicechassis.*` (diverso da `servicetemplate.*`).

**Struttura del chassis:**

```
service-chassis/
├── service-chassis-dependencies-bom     # Bill of Materials delle versioni
├── service-chassis-bom                  # BOM del chassis (aggrega tutti i subproject)
├── service-chassis-util                 # Jackson ObjectMapper con KotlinModule
├── service-chassis-domain-security      # AuthenticatedUser, AuthenticatedUserSupplier
├── service-chassis-persistence          # spring-boot-starter-data-jpa + MySQL driver
├── service-chassis-web                  # spring-boot-starter-web
├── service-chassis-web-swagger          # springdoc-openapi-ui
├── service-chassis-web-security         # SecurityConfig OAuth2/JWT + DefaultAuthenticatedUserSupplier
├── service-chassis-health-check         # spring-boot-starter-actuator
├── service-chassis-metrics              # Micrometer + Prometheus + MetricsConfiguration
├── service-chassis-distributed-tracing  # Spring Cloud Sleuth + Zipkin
├── service-chassis-test-containers      # KeyCloakContainer, MySqlContainer, ZipkinContainer
├── service-chassis-test-keycloak        # JwtProvider per test
└── service-chassis-test-util            # Eventually helper
```

**`service-chassis-dependencies-bom`** — centralizza tutte le versioni:

```kotlin
// build.gradle.kts del dependencies-bom
javaPlatform { allowDependencies() }

dependencies {
    api(platform(org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES))
    api(platform("org.springframework.cloud:spring-cloud-dependencies:2021.0.0"))

    constraints {
        api("org.testcontainers:testcontainers:1.16.2")
        api("io.micrometer:micrometer-registry-prometheus:1.6.4")
        api("org.keycloak:keycloak-admin-client:17.0.1")
        api("io.rest-assured:rest-assured:4.3.3")
        // ...
    }
}
```

**`service-chassis-bom`** — aggrega tutti i moduli del chassis come constraints:

```kotlin
javaPlatform {
    allowDependencies()
}
dependencies {
    api(platform(project(":service-chassis-dependencies-bom")))
    // tutti i subproject diventano constraints versionati
    constraints {
        rootProject.subprojects.filter { !it.name.endsWith("-bom") }.forEach { api(it) }
    }
}
```

**`service-chassis-domain-security`** — isola `AuthenticatedUser` e `AuthenticatedUserSupplier` come contratti del dominio:

```kotlin
// AuthenticatedUser.kt — contratto del dominio, nessuna dipendenza Spring Security
interface AuthenticatedUserSupplier : Supplier<AuthenticatedUser> {
    object EMPTY_SUPPLIER : AuthenticatedUserSupplier {
        override fun get() = AuthenticatedUser("nullId", emptySet())
    }
}

data class AuthenticatedUser(val id: String, val roles: Set<String>)
```

**`service-chassis-web-security`** — `SecurityConfig` parametrizzata con `service.template.role`:

```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig(
    @Value("\${service.template.role}") val serviceRole: String  // configurabile!
) : WebSecurityConfigurerAdapter() {

    override fun configure(http: HttpSecurity) {
        http
            .authorizeRequests()
            .antMatchers("/actuator/**").permitAll()
            .antMatchers("/swagger**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
            .antMatchers("/**").hasRole(serviceRole)  // ruolo configurabile via properties
            .and()
            .oauth2ResourceServer().jwt { it.jwtAuthenticationConverter(jwtAuthenticationConverter()) }
    }

    @Bean
    fun authenticatedUserSupplier(): AuthenticatedUserSupplier = DefaultAuthenticatedUserSupplier()
}
```

**Perché `service.template.role` è configurabile?** Ogni servizio che usa il chassis può avere il proprio ruolo Keycloak. Il chassis non hard-coda il ruolo.

Il `service-template` ora dipende dal chassis via `build.gradle.kts` del modulo `service-template-web-security`:

Nell'`application.properties` finale:

```properties
service.template.role=service-template-user
service.template.controller.base.package=net.chrisrichardson.liveprojects.servicetemplate.web
```

### Milestone 2 — Separazione fisica in due Gradle project

La milestone 2 è una ristrutturazione fondamentale: `service-chassis` e `service-template` diventano **due progetti Gradle completamente separati**, ciascuno con il proprio `settings.gradle.kts`, `build.gradle.kts` root e `gradlew`.

**Prima (milestone 1):** tutto in un unico Gradle project con `settings.gradle.kts` che include sia `service-chassis-*` che `service-template-*`.

**Dopo (milestone 2):**

```
/04-chassis/milestone-2/
├── service-chassis/             # Gradle project autonomo
│   ├── settings.gradle.kts      # rootProject.name = "live-projects-service-chassis"
│   ├── build.gradle.kts         # subprojects: pubblicazione Maven locale
│   └── service-chassis-*/
└── service-template/            # Gradle project autonomo
    ├── settings.gradle.kts      # rootProject.name = "live-projects-service-template"
    ├── build.gradle.kts         # dipende da service-chassis via Maven
    └── (domain, web, persistence, metrics, main, ...)
```

Il `service-chassis/build.gradle.kts` aggiunge la pubblicazione Maven:

```kotlin
subprojects {
    apply(plugin = "maven-publish")
    configure<PublishingExtension> {
        publications {
            create<MavenPublication>("myLibrary") {
                if (project.name.endsWith("-bom"))
                    from(components["javaPlatform"])
                else
                    from(components["java"])
            }
        }
        repositories {
            maven {
                name = "myRepo"
                url = uri(File(rootDir, "../build/repository"))
                // pubblica in un repository Maven locale su filesystem
            }
        }
    }
}
```

Il `service-template/build.gradle.kts` risolve il chassis come dipendenza Maven:

```kotlin
subprojects {
    repositories {
        mavenCentral()
        maven(url = "${rootDir}/../build/repository")  // repository locale del chassis
    }
    dependencies {
        implementation(platform("net.chrisrichardson.liveprojects.servicechassis:service-chassis-bom:0.0.1-SNAPSHOT"))
    }
}
```

I moduli del service-template ora hanno `build.gradle.kts` minimali:

```kotlin
// service-template/domain/build.gradle.kts
apply<IntegrationTestsPlugin>()
allOpen {
    annotation("javax.persistence.Entity")
    annotation("javax.persistence.Embeddable")
    annotation("javax.persistence.MappedSuperclass")
}
dependencies {
    implementation("net.chrisrichardson.liveprojects.servicechassis:service-chassis-domain-security")
    api("org.springframework.data:spring-data-commons")
    compileOnly("org.springframework.boot:spring-boot-starter-data-jpa")
    testImplementation(project(":service-template-test-data"))
}
```

### Milestone 3 — Gradle plugins pubblicati come libreria

La milestone 3 introduce `service-chassis-plugins`: un modulo Gradle che pubblica plugin riusabili per gestire le dipendenze di ogni tipo di modulo (domain, web, persistence, metrics, main, web-security).

**Struttura del plugin:**

```
service-chassis-plugins/
├── src/main/kotlin/
│   ├── IntegrationTestsPlugin.kt     # aggiunge il sourceSet integrationTest
│   ├── ComponentTestsPlugin.kt       # aggiunge il sourceSet componentTest
│   └── RestAssuredTestDependenciesPlugin.kt
└── src/main/groovy/
    ├── ServiceProjectPlugin.groovy       # plugin radice: configura Kotlin, docker-compose, BOM
    ├── ServiceDomainModulePlugin.groovy  # dipendenze per il modulo domain
    ├── ServiceWebModulePlugin.groovy     # dipendenze per il modulo web
    ├── ServicePersistenceModulePlugin.groovy
    ├── ServiceWebSecurityModulePlugin.groovy
    ├── ServiceMetricsModulePlugin.groovy
    └── ServiceMainModulePlugin.groovy    # assembla tutto il servizio
```

**Nota tecnica:** I plugin più complessi (che usano l'API Groovy DSL di Gradle) sono scritti in Groovy, mentre quelli più semplici (come `IntegrationTestsPlugin`) sono in Kotlin. Questa scelta è pratica: l'API Gradle Groovy DSL è più matura per certi pattern.

**`IntegrationTestsPlugin.kt`** — crea il sourceSet `integrationTest` come custom test suite:

```kotlin
class IntegrationTestsPlugin : Plugin<Project> {
    override fun apply(project: Project) {
        val sourceSets = project.the<SourceSetContainer>()
        val main = sourceSets.findByName("main")!!
        val test = sourceSets.findByName("test")!!

        val integrationTestSourceSet = sourceSets.create("integrationTest") {
            compileClasspath += main.output + test.output
            runtimeClasspath += main.output + test.output
        }

        // integrationTestImplementation estende testImplementation
        configurations["integrationTestImplementation"]
            .extendsFrom(configurations.findByName("testImplementation"))
        configurations["integrationTestRuntimeOnly"]
            .extendsFrom(configurations.findByName("runtimeOnly"))

        val integrationTest = project.task<Test>("integrationTest") {
            description = "Runs integration tests."
            group = "verification"
            testClassesDirs = integrationTestSourceSet.output.classesDirs
            classpath = integrationTestSourceSet.runtimeClasspath
            shouldRunAfter("test")
        }

        // integrationTest viene eseguito quando si fa ./gradlew check
        project.tasks.findByName("check")!!.dependsOn(integrationTest)
    }
}
```

**Perché un sourceSet separato?** I test di integrazione richiedono infrastruttura (database, Keycloak, Zipkin via Testcontainers). Separarli permette di eseguire `./gradlew test` (solo unit test, veloci) separatamente da `./gradlew integrationTest` (che avvia i container). Entrambi sono inclusi in `./gradlew check`.

**`ServiceProjectPlugin.groovy`** — plugin root applicato al `build.gradle.kts` principale del service-template:

```groovy
class ServiceProjectPlugin implements Plugin<Project> {
    void apply(Project rootProject) {
        // Configura docker-compose plugin (infrastruttura locale)
        rootProject.apply plugin: "docker-compose"
        rootProject.dockerCompose {
            infrastructure {
                startedServices = ["mysql", "keycloak"]
            }
        }

        // Configura tutti i subproject
        rootProject.subprojects { project ->
            // Kotlin compiler options
            tasks.withType(KotlinCompile) {
                kotlinOptions { freeCompilerArgs = ["-Xjsr305=strict"]; jvmTarget = "11" }
            }
            tasks.withType(Test) { useJUnitPlatform() }

            // Plugin Kotlin standard
            project.apply plugin: "org.jetbrains.kotlin.jvm"
            project.apply plugin: "org.jetbrains.kotlin.plugin.spring"
            project.apply plugin: "org.jetbrains.kotlin.plugin.jpa"
            project.apply plugin: "org.jetbrains.kotlin.plugin.allopen"

            // Dipendenze comuni a tutti i subproject
            project.dependencies {
                implementation(platform("net.chrisrichardson.liveprojects.servicechassis:service-chassis-bom:0.0.1-SNAPSHOT"))
                implementation("org.jetbrains.kotlin:kotlin-reflect")
                implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
                testImplementation("org.springframework.boot:spring-boot-starter-test")
                testImplementation("org.mockito.kotlin:mockito-kotlin")
            }
        }

        // bootRun dipende dall'infrastruttura docker
        rootProject.tasks.findByPath(":main:bootRun").dependsOn(":infrastructureComposeUp")
    }
}
```

**`ServiceMainModulePlugin.groovy`** — plugin per il modulo `main`:

```groovy
class ServiceMainModulePlugin implements Plugin<Project> {
    void apply(Project p) {
        p.apply plugin: "org.springframework.boot"
        // applica IntegrationTestsPlugin, ComponentTestsPlugin, RestAssuredTestDependenciesPlugin

        p.bootJar { layered { enabled = true } }  // layered jar

        p.dependencies {
            // dipende dai moduli locali del servizio
            implementation(project(path: ":domain"))
            implementation(project(path: ":config"))
            implementation(project(path: ":persistence"))
            implementation(project(path: ":web"))
            implementation(project(path: ":web-security"))
            implementation(project(path: ":metrics"))

            // dipende dal chassis via Maven coordinate
            implementation("net.chrisrichardson.liveprojects.servicechassis:service-chassis-util")
            implementation("net.chrisrichardson.liveprojects.servicechassis:service-chassis-distributed-tracing")
            implementation("net.chrisrichardson.liveprojects.servicechassis:service-chassis-web-swagger")
            implementation("net.chrisrichardson.liveprojects.servicechassis:service-chassis-health-check")
            // ...
        }
    }
}
```

Il `build.gradle.kts` del modulo `main` del service-template diventa minimo:

```kotlin
// service-template/main/build.gradle.kts — finale, con plugin
plugins {
    id("net.chrisrichardson.liveprojects.servicechassis.plugins.ServiceMainModulePlugin")
}
```

Analogamente per tutti gli altri moduli:

```kotlin
// domain/build.gradle.kts
plugins {
    id("net.chrisrichardson.liveprojects.servicechassis.plugins.ServiceDomainModulePlugin")
}

// web/build.gradle.kts
plugins {
    id("net.chrisrichardson.liveprojects.servicechassis.plugins.ServiceWebModulePlugin")
}

// persistence/build.gradle.kts
plugins {
    id("net.chrisrichardson.liveprojects.servicechassis.plugins.ServicePersistenceModulePlugin")
}

// web-security/build.gradle.kts
plugins {
    id("net.chrisrichardson.liveprojects.servicechassis.plugins.ServiceWebSecurityModulePlugin")
}

// metrics/build.gradle.kts
plugins {
    id("net.chrisrichardson.liveprojects.servicechassis.plugins.ServiceMetricsModulePlugin")
}
```

**Vantaggi:** Un nuovo servizio che usa il chassis ha `build.gradle.kts` di 2-3 righe per modulo. Tutta la complessità delle dipendenze e dei task è centralizzata nel chassis.

### Milestone 4 — Versioning del plugin con properties file embedded

Il plugin `ServiceProjectPlugin` nella milestone 3 aveva la versione del BOM hard-codata. La milestone 4 risolve questo con un meccanismo di self-versioning:

```kotlin
// CreateServicePluginPropertiesFileTask.kt — genera un file .properties all'interno del JAR
abstract class CreateServicePluginPropertiesFileTask : DefaultTask() {

    @OutputFile
    val propFile = project.file("${project.buildDir}/generated/service.plugin.properties")

    @Input
    var pluginVersion: String = ""

    @Input
    var chassisRepo: String = ""

    @TaskAction
    fun generateFile() {
        project.mkdir(propFile.parent)
        propFile.writeText("""
version=$pluginVersion
chassisRepo=$chassisRepo
        """)
    }
}
```

Il `service-chassis-plugins/build.gradle.kts` registra e usa questo task:

```kotlin
val createPropsFileTask = tasks.register<CreateServicePluginPropertiesFileTask>(
    "createServicePluginPropertiesFile"
) {
    pluginVersion = project.version as String
    chassisRepo = if (project.hasProperty("chassisRepo"))
        project.property("chassisRepo") as String else "local"
}

val processResourcesTask: Copy = tasks.findByPath("processResources") as Copy
processResourcesTask.from(files(createPropsFileTask))  // include nel JAR
```

Il `ServiceProjectPlugin.groovy` legge queste properties al runtime:

```groovy
class ServiceProjectPlugin implements Plugin<Project> {
    void apply(Project rootProject) {
        def props = new Properties()
        props.load(getClass().getResourceAsStream("/service.plugin.properties"))

        def thisVersion = props.getProperty("version")
        def thisChassisRepo = props.getProperty("chassisRepo")

        // Se "local", punta al repository su filesystem locale
        if (thisChassisRepo == "local")
          thisChassisRepo = new File(rootProject.rootDir, "../build/repository")

        // Usa la versione letta dal file per il BOM
        project.dependencies {
            implementation(platform(
                "net.chrisrichardson.liveprojects.servicechassis:service-chassis-bom:$thisVersion"
            ))
        }
    }
}
```

**Perché questo meccanismo?** Il plugin JAR deve sapere quale versione del chassis usare senza che il consumatore debba specificarla. Il file `.properties` viene embedded nel JAR del plugin durante il build, contenendo la versione corrente. Così quando un servizio usa il plugin, ottiene automaticamente la versione del chassis corrispondente.

Supporta anche repository remoti tramite variabili d'ambiente:

```groovy
project.repositories {
    mavenCentral()
    maven {
        url = uri(thisChassisRepo)
        if (System.getenv("MAVEN_REPO_USERNAME") != null)
            credentials {
                username = System.getenv("MAVEN_REPO_USERNAME")
                password = System.getenv("MAVEN_REPO_PASSWORD")
            }
    }
}
```

### Milestone 5 — Pipeline CI/CD completa e pubblicazione del service-template

La milestone finale aggiunge:

1. **Pipeline CI/CD che costruisce e pubblica tutto** (chassis + immagine Docker + service-template come branch Git)
2. **Script `publish-service-template.sh`** per pubblicare il service-template su un branch Git separato

**`.github/workflows/build.yml` — pipeline finale:**

```yaml
- name: Build and publish chassis
  run: |
    cd service-chassis
    ./gradlew build
    ./gradlew publish  # pubblica su Maven locale

- name: Build service template
  run: |
    cp dot.testcontainers.properties ~/.testcontainers.properties
    cd service-template
    ./gradlew compileAll
    export DOCKER_HOST_IP=$(hostname -I | sed -e 's/ .*//g')
    ./gradlew build    # include unit + integration + component test

- name: Publish chassis
  run: |
    cd service-chassis
    export MAVEN_REPO_USERNAME=cer
    export MAVEN_REPO_PASSWORD="${{ secrets.GITHUB_TOKEN }}"
    ./gradlew publish -P chassisRepo=https://maven.pkg.github.com/$GITHUB_REPOSITORY

- name: Publish template
  run: |
    ./publish-service-template.sh "https://maven.pkg.github.com/$GITHUB_REPOSITORY"

- name: Publish image
  run: |
    docker tag live-projects-service-template ghcr.io/$GITHUB_REPOSITORY
    docker push ghcr.io/$GITHUB_REPOSITORY
```

**`publish-service-template.sh`** — pubblica il service-template come branch Git (`published-service-template`):

```bash
# Crea o aggiorna il branch published-service-template come Git worktree
git worktree add -B published-service-template $THE_DIR origin/published-service-template

# Copia i file del service-template (esclusi i riferimenti al chassis locale)
cp -r $ROOT/service-template/* .
rm -fr service-chassis-*

# Aggiorna settings.gradle.kts con il repo Maven remoto
sed -i.bak -e '/service-chassis/d' \
    -e "s?^.*MAVEN_REPO_URL.*\$?          url = uri(\"${1}\")?" \
    settings.gradle.kts

git commit -am "Updated"
git push
```

**Perché un branch Git separato?** Il service-template completo (pronto per essere clonato da altri team) viene pubblicato su un branch separato del repository. I consumatori possono usare `git clone --branch published-service-template` o semplicemente scaricare il contenuto. Il `settings.gradle.kts` pubblicato punta al Maven Package Registry di GitHub, non al filesystem locale.

Il `settings.gradle.kts` del service-template in questa milestone usa `pluginManagement` per risolvere il plugin dal repository Maven:

```kotlin
// service-template/settings.gradle.kts
pluginManagement {
    repositories {
        maven {
            url = uri(System.getenv("MAVEN_REPO_URL") ?: File(rootDir, "../build/repository"))
            if (System.getenv("MAVEN_REPO_USERNAME") != null)
                credentials {
                    username = System.getenv("MAVEN_REPO_USERNAME")
                    password = System.getenv("MAVEN_REPO_PASSWORD")
                }
        }
    }
}
```

---

## Pattern e decisioni di design notevoli

### 1. Observer pattern per il disaccoppiamento del dominio dalle metriche

Il dominio definisce `AccountServiceObserver` come interfaccia pura. `MicrometerBasedAccountServiceObserver` implementa questa interfaccia nel modulo `metrics`. Il dominio è testabile senza Micrometer nel classpath. In un test unitario si può passare `null` o uno stub:

```kotlin
// Test unitario: nessuna dipendenza Micrometer
val service = AccountServiceImpl(
    accountRepository = mockRepository,
    authenticatedUserSupplier = AuthenticatedUserSupplier.EMPTY_SUPPLIER,
    accountServiceObserver = null  // oppure un mock
)
```

### 2. Sealed class per i risultati delle operazioni di dominio

Invece di lanciare eccezioni o usare `Optional`, il progetto usa sealed class per ogni possibile esito. Questo rende espliciti tutti i casi nel chiamante e permette di usare `when` exhaustive:

```kotlin
when (val outcome = accountService.createAccount(request.initialBalance)) {
    is Success -> ResponseEntity.ok(AccountDTO(outcome.account.id!!, ...))
    is AmountNotGreaterThanZero -> ResponseEntity(ErrorResponse("..."), HttpStatus.CONFLICT)
    else -> ResponseEntity(ErrorResponse("Unexpected: ${outcome::class.simpleName}"), INTERNAL_SERVER_ERROR)
}
```

### 3. IntegrationTestsPlugin: separazione dei test per livello

Il plugin crea un sourceSet `integrationTest` separato da `test`. I test di integrazione richiedono infrastruttura (Testcontainers), quelli unitari no. Il workflow è:

- `./gradlew test` — solo unit test (secondi)
- `./gradlew integrationTest` — avvia container + esegui integration test (minuti)
- `./gradlew check` — entrambi in sequenza (`test` poi `integrationTest`)
- `./gradlew componentTest` — test end-to-end con docker-compose

### 4. BOM (Bill of Materials) a doppio livello

Il chassis usa due BOM separati:
- `service-chassis-dependencies-bom`: centralizza le versioni delle dipendenze esterne (Testcontainers, Spring Boot, Spring Cloud, Micrometer, ecc.)
- `service-chassis-bom`: aggrega tutti i moduli del chassis come constraints versionati

I consumatori importano solo `service-chassis-bom` e ottengono sia la gestione delle versioni dei moduli chassis che quella delle dipendenze esterne.

### 5. Null Object pattern per AuthenticatedUserSupplier

```kotlin
interface AuthenticatedUserSupplier : Supplier<AuthenticatedUser> {
    object EMPTY_SUPPLIER : AuthenticatedUserSupplier {
        override fun get() = AuthenticatedUser("nullId", emptySet())
    }
}
```

Evita null checks ovunque nel dominio. Se la sicurezza non è configurata (es. nei test del solo dominio), viene usato automaticamente `EMPTY_SUPPLIER`.

### 6. Testcontainers con `withReuse(true)` e oggetti singleton

I container Keycloak, MySQL e Zipkin sono definiti come `object` Kotlin (singleton):

```kotlin
object KeyCloakContainer : DefaultPropertyProvidingContainer() { ... }
object MySqlContainer : DefaultPropertyProvidingContainer() { ... }
object ZipkinContainer : DefaultPropertyProvidingContainer() { ... }
```

Il singleton garantisce che lo stesso container sia riusato da tutti i test nella stessa JVM. Con `withReuse(true)` il container sopravvive anche tra JVM diverse (identificato dal hash della configurazione).

### 7. Plugin Gradle con versioning self-contained

Il `service-chassis-plugins` JAR contiene un file `service.plugin.properties` generato durante il build con la propria versione e il repository URL. Il `ServiceProjectPlugin` lo legge a runtime per configurare le dipendenze BOM. Questo elimina il bisogno che i consumatori specifichino versioni nelle loro dipendenze.

### 8. Docker layered jar per build cache efficiente

```
dependencies/          # raramente cambia → strato Docker stabile
snapshot-dependencies/ # cambia raramente
spring-boot-loader/    # quasi mai cambia
application/           # cambia ad ogni build
```

In una pipeline CI tipica, solo l'ultimo layer viene ricostruito. I layer delle dipendenze vengono serviti dalla cache Docker locale o dalla registry.

### 9. Component test con `Wait.forHealthcheck()`

Il component test aspetta che il HEALTHCHECK del Dockerfile sia verde prima di eseguire i test. Questo è più affidabile di aspettare su una porta TCP: verifica che l'applicazione sia effettivamente funzionante, non solo avviata.

### 10. publish-service-template.sh: Git worktree per il template pubblicato

Il service-template viene pubblicato su un branch Git separato (`published-service-template`) che contiene solo il codice del servizio, senza il chassis. I riferimenti locali al chassis sono sostituiti da coordinate Maven remote. Questo permette ad altri team di clonare direttamente il branch come punto di partenza.

---

## Risorse e riferimenti

- **Chris Richardson** — microservices.io, autore di *Microservices Patterns* (Manning)
- **Spring Cloud Sleuth** — distributed tracing autoconfigure per Spring Boot
- **Micrometer** — facade per metriche, con backend Prometheus
- **Testcontainers** — libreria per Docker-based testing in JVM
- **Keycloak** — Identity Provider open source, usato qui come Authorization Server OAuth2
- **Spring Security OAuth2 Resource Server** — valida JWT emessi da Keycloak
- **Spring Boot layered JAR** — `java -Djarmode=layertools`
- **Gradle BOM** (`java-platform` plugin) — gestione versioni centralizzata
- **Gradle Plugin development** — `gradle-plugin` + `gradlePlugin {}` DSL
- **GitHub Packages** — Maven Package Registry per pubblicazione del chassis
- **Docker HEALTHCHECK** — usato in combinazione con Testcontainers `Wait.forHealthcheck()`

---

## Takeaway applicabili

1. **Il chassis è il vero prodotto.** Il service-template è solo un esempio/demo del chassis. Il valore è nei moduli `service-chassis-*` riusabili.

2. **Usare sealed class per i risultati di dominio** invece di eccezioni. Rende tutti i casi gestibili a compile time e forza il chiamante a gestire ogni scenario.

3. **Separare i test per livello** con sourceSet dedicati (`test`, `integrationTest`, `componentTest`). Non mischiare test veloci con test che richiedono infrastruttura.

4. **Testcontainers singleton con `withReuse(true)`** riduce drasticamente i tempi di test: Keycloak impiega 30+ secondi per avviarsi. Con il reuse, viene avviato una volta e riutilizzato.

5. **Il pattern Observer** (interfaccia nel dominio, implementazione nel modulo metrics) mantiene il dominio libero da dipendenze di infrastruttura e testabile in isolamento.

6. **Il BOM a doppio livello** (dependencies-bom + chassis-bom) è la soluzione corretta per gestire versioni in un progetto multi-modulo distribuito. I consumatori importano solo il BOM finale.

7. **Il Dockerfile multi-stage con layered JAR** è la best practice per immagini Docker di Spring Boot: riduce il tempo di build e il consumo di banda in CI/CD.

8. **I Gradle plugin pubblicati** permettono ai team di adottare il chassis con `build.gradle.kts` di 2-3 righe per modulo. Questo è il vero "leverage" del chassis: abbassa drasticamente il costo di setup di un nuovo microservizio.

9. **La pipeline CI/CD pubblica sia il chassis (Maven) che il template (Git branch) che l'immagine Docker.** Questi tre artefatti servono audiences diverse: team di sviluppo, nuovi servizi, operazioni.

10. **La property `service.template.role`** rende il chassis riusabile in contesti diversi. Ogni servizio può avere il proprio ruolo Keycloak senza modificare il chassis.

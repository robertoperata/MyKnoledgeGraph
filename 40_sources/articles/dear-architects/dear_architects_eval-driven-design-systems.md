---
tags:
  - design-system
  - ai-coding-assistant
  - eval-driven-development
  - typescript
  - zod
  - llm-tooling
feature:
type: article
author: Jay Johnson
source: https://engineering.gusto.com/eval-driven-design-systems-8f781dc2dacb
date: 2026-08-02
---

# Eval-Driven Design Systems (Part 1)

## Sunto

L'articolo affronta un problema emergente nell'adozione massiva degli assistenti AI per lo sviluppo UI: i modelli linguistici addestrati su dati pubblici (Material-UI, Chakra, Mantine, Tailwind) non hanno mai incontrato i design system privati aziendali, e di conseguenza li ignorano completamente. Quando un ingegnere chiede all'AI di generare componenti, il modello produce una media delle librerie pubbliche più popolari anziché implementazioni corrette del sistema proprietario. L'autore identifica tre errori tipici: l'assistente bypassa il design system usando HTML grezzo e Tailwind, utilizza prop inventate non presenti nello schema reale, o manca le linee guida sui contenuti e i requisiti di accessibilità.

La tesi centrale del team di Gusto è che **"l'AI-leggibilità è oggi un quality bar, esattamente come l'accessibilità"**. Se la più grande categoria di autori UI (gli assistenti AI) non è in grado di leggere ciò che il design system codifica, il valore del sistema crolla per quell'audience. Questo ha spinto il team a ripensare il modo in cui il design system espone la propria conoscenza, non solo agli ingegneri umani, ma anche ai modelli AI.

La soluzione proposta si basa sull'uso di **Zod schemas come contratti leggibili dagli agenti**. Anziché affidarsi esclusivamente ai tipi TypeScript — che i modelli interpretano in modo generico — ogni componente viene arricchito con metadati strutturati: esempi di codice reale e validato che il modello può copiare direttamente, linee guida sui contenuti (regole di voice, limiti di caratteri), requisiti di accessibilità (pattern ARIA, comportamento per screen reader), import statement esatti per prevenire submodule allucinati, e URL Figma per i tool di design downstream. Gli schemi Zod permettono inoltre di codificare combinazioni di prop illegali come union discriminate, prevenendo a livello di schema le combinazioni non supportate.

Il Part 2 dell'articolo (non ancora pubblicato) promette di coprire le metriche di valutazione e la misurazione tramite Braintrust — suggerendo un approccio sistematico e quantitativo per misurare il miglioramento effettivo nella qualità dell'output AI post-intervento. Questo allinea il design system ai principi dell'eval-driven development: definire metriche, misurare baseline, applicare modifiche, misurare di nuovo.

## Codice

Schema Zod per un componente Button che previene combinazioni di prop illegali tramite union discriminata, arricchito con metadati per gli agenti AI:

```typescript
const buttonShared = z.object({
  size: z.enum(['lg', 'md', 'sm']).default('md'),
});

export const buttonSchema = z.union([
  buttonShared.extend({
    variant: z.enum(['primary', 'ghost']).optional(),
    tone: z.enum(['critical', 'neutral']).optional(),
  }).strict(),
  buttonShared.extend({
    variant: z.enum(['secondary', 'outline']).default('secondary'),
    tone: z.literal('critical').optional(),
  }).strict(),
]).meta({
  importStatement: "import { Button } from '@acme/design-system';",
  examples: [
    { code: '<Button variant="secondary">Save changes</Button>' },
  ],
});
```

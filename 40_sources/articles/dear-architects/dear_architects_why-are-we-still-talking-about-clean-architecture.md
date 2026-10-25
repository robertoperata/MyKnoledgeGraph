---
tags:
  - clean-architecture
  - functional-programming
  - software-design
  - hexagonal-architecture
  - domain-driven-design
feature:
type: article
author: Rico Fritzsche
source: https://ricofritzsche.me/why-are-we-still-talking-about-clean-architecture/
date: 2026-10-25
---

# Why Are We Still Talking About Clean Architecture?

## Sunto

Rico Fritzsche apre il suo articolo con una provocazione: dovremmo smettere di parlare di Clean Architecture, Hexagonal Architecture e Onion Architecture — non perché siano sbagliate, ma perché non risolvono il problema fondamentale che dichiarano di voler risolvere. L'**illusione del disaccoppiamento** è il cuore critico dell'articolo: invertire la direzione delle dipendenze nel codice sorgente non significa effettivamente separare la business logic dall'infrastruttura. La logica di business dipende ancora da chiamate esterne che possono fallire e produrre side effect; cambia solo la direzione della freccia nel diagramma.

L'autore sfida anche l'argomento classico a favore di questi approcci — la possibilità di scambiare il database senza toccare il dominio. Nella pratica, le differenze tra database (comportamento transazionale, operatori di query, semantica degli errori) *traspaiono* attraverso qualsiasi astrazione, rendendo lo scambio effettivamente impossibile senza modificare anche il codice di dominio.

La soluzione proposta è il pattern **Functional Core / Imperative Shell**. Il principio è semplice: il *functional core* contiene le regole di business come funzioni pure — ricevono valori in input e restituiscono risultati, senza side effect, senza dipendenze esterne. L'*imperative shell* raccoglie i dati necessari dall'infrastruttura, li passa al core, e persiste il risultato. Questo approccio viene applicato **per capability di dominio**, non per layer applicativo, così ogni capability ha il proprio core e la propria shell.

Il vantaggio principale è la testabilità: il functional core può essere testato con input diretti e output attesi, senza mock, senza container, senza database in memory. Un test per una regola di booking, ad esempio, riceve semplicemente i dati della prenotazione come valori e verifica il risultato restituito — eliminando l'infrastruttura di test tipicamente necessaria con le architetture a layer. L'autore mostra con TypeScript come si passa da un `BookingService` che dipende da un repository a una funzione `evaluateBooking` pura e isolata.

La conclusione è radicale ma coerente: se l'obiettivo è isolare davvero la business logic e renderla testabile, il Functional Core / Imperative Shell raggiunge questo obiettivo molto meglio delle architetture layered tradizionali, eliminando la complessità strutturale delle porte e degli adapter senza perderne i benefici di isolamento.

## Codice

Il pattern `BookingService` con dipendenza da repository (approccio tradizionale):

```typescript
class BookingService {
  constructor(private reservationRepository: ReservationRepository) {}

  async book(reservationData: ReservationData): Promise<Booking> {
    const existing = await this.reservationRepository.find(reservationData);
    // business logic entangled with infrastructure
    if (existing) throw new Error('Already booked');
    return this.reservationRepository.save(new Booking(reservationData));
  }
}
```

La funzione `evaluateBooking` come functional core puro:

```typescript
type BookingResult = { accepted: true; booking: Booking } | { accepted: false; reason: string };

function evaluateBooking(
  reservationData: ReservationData,
  existingReservations: Reservation[]
): BookingResult {
  const conflict = existingReservations.find(r => overlaps(r, reservationData));
  if (conflict) return { accepted: false, reason: 'Slot already taken' };
  return { accepted: true, booking: new Booking(reservationData) };
}
```

Test del functional core senza infrastruttura:

```typescript
test('rejects overlapping reservations', () => {
  const existing = [buildReservation({ slot: '09:00-10:00' })];
  const result = evaluateBooking({ slot: '09:30-10:30' }, existing);
  expect(result.accepted).toBe(false);
});
```

# Appunti di architettura software

Appunti brevi e pratici su architettura software, Domain-Driven Design, CQRS e sistemi distribuiti.

Il punto di partenza è comprendere il problema e le regole di business insieme al cliente e agli utenti. Da qui esaminiamo le decisioni alla base dei pattern architetturali: quali problemi risolvono, quando introducono complessità inutile e quali compromessi comportano.

Questi articoli si rivolgono a sviluppatori esperti, responsabili tecnici, architetti software e responsabili dei team di sviluppo che devono prendere e spiegare decisioni architetturali.

## Approccio

Ogni articolo parte da un problema concreto, considera la soluzione praticabile più semplice e si conclude con una decisione basata su vincoli e compromessi espliciti.

```text
Contesto → Problema → Vincoli → Opzioni → Compromessi → Decisione
```

Gli esempi rimangono piccoli e usano TypeScript, con NestJS e PostgreSQL solo dove aiutano a spiegare la decisione. Il dominio ricorrente è un'applicazione Todo con una regola di business: **un utente non può avere più di tre todo attivi**.

## Capitoli

I dodici capitoli hanno diagrammi ed esempi pratici; la colonna Stato indica quali sono ancora in bozza.

| # | Capitolo | Domanda centrale | Stato |
| --- | --- | --- | --- |
| 1 | [Prima dell'architettura: capire il problema di business](articles/business-before-architecture.md) | Abbiamo compreso il problema e le regole prima di scegliere la tecnologia? | Completato |
| 2 | [Quando basta un monolite modulare](articles/modular-monolith.md) | Ci servono servizi indipendenti o confini più chiari tra moduli? | Completato |
| 3 | [Quando CQRS è eccessivo](articles/cqrs-overkill.md) | Il modello di lettura e scrittura giustifica la complessità aggiuntiva? | Completato |
| 4 | [Gli eventi di dominio non sono eventi di integrazione](articles/domain-vs-integration-events.md) | Un concetto interno al dominio dovrebbe diventare un contratto pubblico? | Completato |
| 5 | [Perché salvare dati e pubblicare un evento è difficile](articles/outbox-pattern.md) | Come colleghiamo una transazione del database a un sistema di messaggistica affidabile? | Completato |
| 6 | [I bounded context sono più di semplici cartelle](articles/bounded-contexts.md) | Il confine del modello è visibile e verificabile nel codice? | Completato |
| 7 | [Il dominio non dovrebbe conoscere HTTP](articles/domain-http-error-mapping.md) | Il dominio avrebbe ancora senso senza HTTP? | Completato |
| 8 | [Testare l'architettura, oltre alla logica di business](articles/architecture-boundary-tests.md) | Come impediamo che i confini architetturali si erodano? | Completato |
| 9 | [Dove dovrebbe finire una transazione?](articles/transactions-eventual-consistency.md) | Quali operazioni richiedono consistenza immediata? | Completato |
| 10 | [I microservizi sono anche una decisione operativa](articles/distributed-systems-cost.md) | Quali benefici giustificano il costo operativo di un sistema distribuito? | Completato |
| 11 | [Dalla palla di fango ai confini espliciti](articles/from-big-ball-of-mud.md) | Da dove si comincia a mettere ordine in un sistema esistente, e fino a dove conviene arrivare? | Bozza |
| 12 | [Il debito tecnico si paga con gli interessi](articles/technical-debt.md) | Quanto debito possiamo permetterci, e chi ne paga gli interessi? | Bozza |

## Roadmap

Articoli pianificati, non ancora scritti. Sono approfondimenti puntuali di temi che i capitoli attuali toccano senza svilupparli.

| Titolo provvisorio | Domanda centrale | Nasce da |
| --- | --- | --- |
| Proteggere una regola sotto concorrenza: lock, versione o `SERIALIZABLE`? | Quale meccanismo di coordinamento costa meno per questa regola? | Capitoli 2 e 9 |
| Provare i guasti: outbox e consumer che si riprendono | Come dimostriamo che il sistema recupera, invece di sperarlo? | Capitoli 5 e 8 |
| Idempotenza: cosa succede se il client riprova? | Possiamo ripetere una richiesta senza duplicarne l'effetto? | Capitoli 5, 9 e 10 |
| Migrazioni senza fermare il servizio | Come cambiamo lo schema mentre due versioni del codice convivono? | Capitolo 10 |
| Serve davvero l'Event Sourcing? | Ci serve la storia come fonte di verità o come registro? | Capitoli 3 e 4 |

## Struttura del repository

```text
articles/
├── business-before-architecture.md       Primo capitolo
├── modular-monolith.md                   Secondo capitolo
├── cqrs-overkill.md                      Terzo capitolo
├── domain-vs-integration-events.md       Quarto capitolo
├── outbox-pattern.md                     Quinto capitolo
├── bounded-contexts.md                   Sesto capitolo
├── domain-http-error-mapping.md          Settimo capitolo
├── architecture-boundary-tests.md        Ottavo capitolo
├── transactions-eventual-consistency.md  Nono capitolo
├── distributed-systems-cost.md           Decimo capitolo
├── from-big-ball-of-mud.md               Undicesimo capitolo
└── technical-debt.md                     Dodicesimo capitolo
examples/                                 Riservata a piccoli esempi autonomi
diagrams/                                 Illustrazioni SVG richiamate negli articoli
```

I primi dieci capitoli seguono un percorso dal problema di business ai costi operativi delle scelte architetturali; l'undicesimo applica quel percorso a un sistema esistente e il dodicesimo spiega come evitare di tornarci. Codice e diagrammi accompagnano il testo quando aiutano a chiarire una specifica decisione.

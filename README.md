# Appunti di architettura software

Appunti brevi e pratici su architettura software, Domain-Driven Design, CQRS e sistemi distribuiti.

L'attenzione è sulle decisioni alla base dei pattern architetturali: quali problemi risolvono, quando introducono complessità inutile e quali compromessi comportano.

Questi articoli si rivolgono a sviluppatori esperti, responsabili tecnici, architetti software e responsabili dei team di sviluppo che devono prendere e spiegare decisioni architetturali.

## Approccio

Ogni articolo parte da un problema concreto, considera la soluzione praticabile più semplice e si conclude con una decisione basata su vincoli e compromessi espliciti.

```text
Contesto → Problema → Vincoli → Opzioni → Compromessi → Decisione
```

Gli esempi rimangono piccoli e usano TypeScript, con NestJS e PostgreSQL solo dove aiutano a spiegare la decisione. Il dominio ricorrente è un'applicazione Todo con una regola di business: **un utente non può avere più di tre todo attivi**.

## Capitoli

Il primo capitolo è disponibile, con diagrammi e un esempio pratico; gli altri capitoli sono in programma.

| # | Capitolo | Domanda centrale | Stato |
| --- | --- | --- | --- |
| 1 | [Quando basta un monolite modulare](articles/modular-monolith.md) | Ci servono servizi indipendenti o confini più chiari tra moduli? | Completato |
| 2 | Quando CQRS è eccessivo | Il modello di lettura e scrittura giustifica la complessità aggiuntiva? | In programma |
| 3 | Gli eventi di dominio non sono eventi di integrazione | Un concetto interno al dominio dovrebbe diventare un contratto pubblico? | In programma |
| 4 | Perché salvare dati e pubblicare un evento è difficile | Come colleghiamo una transazione del database a un sistema di messaggistica affidabile? | In programma |
| 5 | I bounded context sono più di semplici cartelle | Il confine del modello è visibile e verificabile nel codice? | In programma |
| 6 | Il dominio non dovrebbe conoscere HTTP | Il dominio avrebbe ancora senso senza HTTP? | In programma |
| 7 | Testare l'architettura, oltre alla logica di business | Come impediamo che i confini architetturali si erodano? | In programma |
| 8 | Dove dovrebbe finire una transazione? | Quali operazioni richiedono consistenza immediata? | In programma |
| 9 | I microservizi sono anche una decisione operativa | Quali benefici giustificano il costo operativo di un sistema distribuito? | In programma |

## Struttura del repository

```text
articles/
└── modular-monolith.md    Primo capitolo
examples/                 Riservata a piccoli esempi autonomi
diagrams/                 Illustrazioni SVG richiamate negli articoli
```

I capitoli vengono sviluppati progressivamente. Codice e diagrammi accompagnano il testo quando aiutano a chiarire una specifica decisione architetturale.

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

I primi dodici capitoli hanno diagrammi ed esempi pratici; dal tredicesimo in poi i capitoli applicano lo stesso metodo all'uso di modelli linguistici nelle decisioni architetturali. La colonna Stato indica quali sono ancora in bozza.

| # | Capitolo | Domanda centrale | Stato |
| --- | --- | --- | --- |
| 1 | [Prima dell'architettura: capire il problema di business](articles/01-business-before-architecture.md) | Abbiamo compreso il problema e le regole prima di scegliere la tecnologia? | Completato |
| 2 | [Quando basta un monolite modulare](articles/02-modular-monolith.md) | Ci servono servizi indipendenti o confini più chiari tra moduli? | Completato |
| 3 | [Quando CQRS è eccessivo](articles/03-cqrs-overkill.md) | Il modello di lettura e scrittura giustifica la complessità aggiuntiva? | Completato |
| 4 | [Gli eventi di dominio non sono eventi di integrazione](articles/04-domain-vs-integration-events.md) | Un concetto interno al dominio dovrebbe diventare un contratto pubblico? | Completato |
| 5 | [Perché salvare dati e pubblicare un evento è difficile](articles/05-outbox-pattern.md) | Come colleghiamo una transazione del database a un sistema di messaggistica affidabile? | Completato |
| 6 | [I bounded context sono più di semplici cartelle](articles/06-bounded-contexts.md) | Il confine del modello è visibile e verificabile nel codice? | Completato |
| 7 | [Il dominio non dovrebbe conoscere HTTP](articles/07-domain-http-error-mapping.md) | Il dominio avrebbe ancora senso senza HTTP? | Completato |
| 8 | [Testare l'architettura, oltre alla logica di business](articles/08-architecture-boundary-tests.md) | Come impediamo che i confini architetturali si erodano? | Completato |
| 9 | [Dove dovrebbe finire una transazione?](articles/09-transactions-eventual-consistency.md) | Quali operazioni richiedono consistenza immediata? | Completato |
| 10 | [I microservizi sono anche una decisione operativa](articles/10-distributed-systems-cost.md) | Quali benefici giustificano il costo operativo di un sistema distribuito? | Completato |
| 11 | [Dalla palla di fango ai confini espliciti](articles/11-from-big-ball-of-mud.md) | Da dove si comincia a mettere ordine in un sistema esistente, e fino a dove conviene arrivare? | Bozza |
| 12 | [Il debito tecnico si paga con gli interessi](articles/12-technical-debt.md) | Quanto debito possiamo permetterci, e chi ne paga gli interessi? | Bozza |
| 13 | [Il dominio non dovrebbe conoscere il modello](articles/13-domain-llm-boundary.md) | Se l'LLM sparisse domani, il caso d'uso avrebbe ancora senso? Dove vive l'adapter? | Bozza |
| 14 | [Una chiamata al modello è una chiamata remota](articles/14-llm-call-is-remote-call.md) | Timeout, retry, idempotenza e costi in token: quale parte della resilienza già nota si applica? | Bozza |
| 15 | [L'output del modello è input non fidato](articles/15-llm-output-untrusted-input.md) | Validiamo la risposta del modello come validiamo una richiesta HTTP? Dove fermiamo una prompt injection? | Bozza |

## Roadmap

Articoli pianificati, non ancora scritti. Sono approfondimenti puntuali di temi che i capitoli attuali toccano senza svilupparli.

| Titolo provvisorio | Domanda centrale | Nasce da |
| --- | --- | --- |
| Proteggere una regola sotto concorrenza: lock, versione o `SERIALIZABLE`? | Quale meccanismo di coordinamento costa meno per questa regola? | Capitoli 2 e 9 |
| Provare i guasti: outbox e consumer che si riprendono | Come dimostriamo che il sistema recupera, invece di sperarlo? | Capitoli 5 e 8 |
| Idempotenza: cosa succede se il client riprova? | Possiamo ripetere una richiesta senza duplicarne l'effetto? | Capitoli 5, 9 e 10 |
| Migrazioni senza fermare il servizio | Come cambiamo lo schema mentre due versioni del codice convivono? | Capitolo 10 |
| Serve davvero l'Event Sourcing? | Ci serve la storia come fonte di verità o come registro? | Capitoli 3 e 4 |

### Temi attuali di architettura

| Titolo provvisorio | Domanda centrale | Nasce da |
| --- | --- | --- |
| Postgres come coda: quando basta `SKIP LOCKED` | Ci serve un broker o ci basta una tabella con un worker? | Capitoli 5 e 10 |
| Esecuzione durevole: saga scritte a mano o Temporal/Restate? | Chi tiene traccia di un processo che dura giorni, e a quale costo operativo? | Capitoli 9 e 10 |
| Timeout, retry e circuit breaker: la resilienza si decide ai confini | Cosa succede al caso d'uso quando la dipendenza esterna rallenta invece di fallire? | Capitoli 7 e 10 |
| Osservabilità come decisione architetturale, non come strumento | Riusciamo a seguire una richiesta attraverso moduli, outbox e consumer senza indovinare? | Capitoli 5 e 10 |
| Contratti guidati dal consumatore: versionare un evento di integrazione | Chi si rompe se cambiamo lo schema di `TodoCompleted`, e come lo scopriamo prima del deploy? | Capitoli 4 e 6 |
| Fitness function: l'architettura evolutiva si misura | Quali proprietà del sistema vogliamo difendere con un test automatico, oltre alle dipendenze? | Capitolo 8 |
| Topologie di team e carico cognitivo: Conway letto al contrario | I confini dei moduli seguono i team o li costringono? | Capitoli 2 e 6 |
| Multi-tenancy: schema condiviso, schema per tenant o database per tenant? | Quanto isolamento ci serve davvero, e quanto costa in migrazioni e operazioni? | Capitoli 9 e 10 |
| Feature flag come confine temporale | Come rilasciamo una regola nuova a metà degli utenti senza due versioni del dominio? | Capitoli 2 e 12 |
| Il costo del cloud come vincolo architetturale | Serverless, container o un'unica VM: quale forma di deploy paga meno per questo carico? | Capitolo 10 |

### L'AI nelle pratiche di architettura

Il filo conduttore è quello dei capitoli dal tredicesimo al quindicesimo: il dominio non conosce HTTP, e non deve conoscere nemmeno il modello. Gli articoli trattano l'AI solo dove cambia una decisione architetturale.

| Titolo provvisorio | Domanda centrale | Nasce da |
| --- | --- | --- |
| Esporre i casi d'uso come tool: l'agente chiama l'application service, non il database | Se un agente crea un todo, chi fa rispettare il limite dei tre attivi? | Capitoli 2 e 6 |
| Serve davvero un vector database? | `pgvector` nel database che abbiamo o un servizio dedicato: quale complessità giustifica il secondo? | Capitoli 3 e 10 |
| Gli eval sono i test di un confine non deterministico | Come dimostriamo che una funzionalità basata su LLM non è peggiorata dopo un cambio di prompt o di modello? | Capitolo 8 |
| Prompt e versione del modello sono dipendenze da versionare | Cambiare prompt è un cambio di contratto? Come lo registriamo e come torniamo indietro? | Capitoli 4 e 12 |
| Human in the loop è un process manager | L'approvazione umana di un'azione proposta dall'agente è un passo di una saga: dove si salva lo stato? | Capitolo 9 |
| I test di architettura come guardrail per il codice generato | Come impediamo che un agente di coding eroda i confini più velocemente di quanto un umano li riveda? | Capitoli 8 e 11 |
| Il debito da comprensione: velocità generata, interessi pagati in revisione | Quanto codice possiamo accettare senza averlo capito, e chi ne paga gli interessi? | Capitolo 12 |
| ADR e documentazione come contesto per gli agenti | Le decisioni scritte per gli umani bastano a un agente, o il repository deve spiegare i propri confini? | Capitoli 1 e 8 |
| L'LLM come consumer di eventi di integrazione | Classificare i todo in modo asincrono a partire da `TodoCreated`: consistenza eventuale con un passo non deterministico | Capitoli 4 e 5 |

## Struttura del repository

```text
articles/
├── 01-business-before-architecture.md       Primo capitolo
├── 02-modular-monolith.md                   Secondo capitolo
├── 03-cqrs-overkill.md                      Terzo capitolo
├── 04-domain-vs-integration-events.md       Quarto capitolo
├── 05-outbox-pattern.md                     Quinto capitolo
├── 06-bounded-contexts.md                   Sesto capitolo
├── 07-domain-http-error-mapping.md          Settimo capitolo
├── 08-architecture-boundary-tests.md        Ottavo capitolo
├── 09-transactions-eventual-consistency.md  Nono capitolo
├── 10-distributed-systems-cost.md           Decimo capitolo
├── 11-from-big-ball-of-mud.md               Undicesimo capitolo
├── 12-technical-debt.md                     Dodicesimo capitolo
├── 13-domain-llm-boundary.md                Tredicesimo capitolo
├── 14-llm-call-is-remote-call.md            Quattordicesimo capitolo
└── 15-llm-output-untrusted-input.md         Quindicesimo capitolo
examples/                                    Riservata a piccoli esempi autonomi
diagrams/                                    Illustrazioni SVG richiamate negli articoli
```

I primi dieci capitoli seguono un percorso dal problema di business ai costi operativi delle scelte architetturali; l'undicesimo applica quel percorso a un sistema esistente e il dodicesimo spiega come evitare di tornarci. I capitoli dal tredicesimo al quindicesimo introducono un modello linguistico nell'applicazione Todo e mostrano che le decisioni restano le stesse: confini espliciti, politiche per le chiamate remote e validazione di ciò che entra. Codice e diagrammi accompagnano il testo quando aiutano a chiarire una specifica decisione.

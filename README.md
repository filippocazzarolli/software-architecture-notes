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

I capitoli sono divisi in due sezioni. La prima segue un percorso dal problema di business ai costi operativi delle scelte architetturali; la seconda applica lo stesso metodo all'uso di modelli linguistici nelle decisioni architetturali. I capitoli pubblicati hanno un diagramma ed esempi pratici. La colonna Stato distingue i capitoli completati, quelli in bozza e quelli di prossima pubblicazione, che non sono ancora scritti e hanno titolo e numero provvisori.

### Sezione 1: architettura e decisioni

| # | Capitolo | Domanda centrale | Stato |
| --- | --- | --- | --- |
| 1.1 | [Prima dell'architettura: capire il problema di business](articles/1.01-business-before-architecture.md) | Abbiamo compreso il problema e le regole prima di scegliere la tecnologia? | Completato |
| 1.2 | [Quando basta un monolite modulare](articles/1.02-modular-monolith.md) | Ci servono servizi indipendenti o confini più chiari tra moduli? | Completato |
| 1.3 | [Quando CQRS è eccessivo](articles/1.03-cqrs-overkill.md) | Il modello di lettura e scrittura giustifica la complessità aggiuntiva? | Completato |
| 1.4 | [Gli eventi di dominio non sono eventi di integrazione](articles/1.04-domain-vs-integration-events.md) | Un concetto interno al dominio dovrebbe diventare un contratto pubblico? | Completato |
| 1.5 | [Perché salvare dati e pubblicare un evento è difficile](articles/1.05-outbox-pattern.md) | Come colleghiamo una transazione del database a un sistema di messaggistica affidabile? | Completato |
| 1.6 | [I bounded context sono più di semplici cartelle](articles/1.06-bounded-contexts.md) | Il confine del modello è visibile e verificabile nel codice? | Completato |
| 1.7 | [Il dominio non dovrebbe conoscere HTTP](articles/1.07-domain-http-error-mapping.md) | Il dominio avrebbe ancora senso senza HTTP? | Completato |
| 1.8 | [Testare l'architettura, oltre alla logica di business](articles/1.08-architecture-boundary-tests.md) | Come impediamo che i confini architetturali si erodano? | Completato |
| 1.9 | [Dove dovrebbe finire una transazione?](articles/1.09-transactions-eventual-consistency.md) | Quali operazioni richiedono consistenza immediata? | Completato |
| 1.10 | [I microservizi sono anche una decisione operativa](articles/1.10-distributed-systems-cost.md) | Quali benefici giustificano il costo operativo di un sistema distribuito? | Completato |
| 1.11 | [Dalla palla di fango ai confini espliciti](articles/1.11-from-big-ball-of-mud.md) | Da dove si comincia a mettere ordine in un sistema esistente, e fino a dove conviene arrivare? | Bozza |
| 1.12 | [Il debito tecnico si paga con gli interessi](articles/1.12-technical-debt.md) | Quanto debito possiamo permetterci, e chi ne paga gli interessi? | Bozza |
| 1.13 | Proteggere una regola sotto concorrenza: lock, versione o `SERIALIZABLE`? | Quale meccanismo di coordinamento costa meno per questa regola? | Prossima pubblicazione |
| 1.14 | Provare i guasti: outbox e consumer che si riprendono | Come dimostriamo che il sistema recupera, invece di sperarlo? | Prossima pubblicazione |
| 1.15 | Idempotenza: cosa succede se il client riprova? | Possiamo ripetere una richiesta senza duplicarne l'effetto? | Prossima pubblicazione |
| 1.16 | Migrazioni senza fermare il servizio | Come cambiamo lo schema mentre due versioni del codice convivono? | Prossima pubblicazione |
| 1.17 | Serve davvero l'Event Sourcing? | Ci serve la storia come fonte di verità o come registro? | Prossima pubblicazione |
| 1.18 | Postgres come coda: quando basta `SKIP LOCKED` | Ci serve un broker o ci basta una tabella con un worker? | Prossima pubblicazione |
| 1.19 | Esecuzione durevole: saga scritte a mano o Temporal/Restate? | Chi tiene traccia di un processo che dura giorni, e a quale costo operativo? | Prossima pubblicazione |
| 1.20 | Timeout, retry e circuit breaker: la resilienza si decide ai confini | Cosa succede al caso d'uso quando la dipendenza esterna rallenta invece di fallire? | Prossima pubblicazione |
| 1.21 | Osservabilità come decisione architetturale, non come strumento | Riusciamo a seguire una richiesta attraverso moduli, outbox e consumer senza indovinare? | Prossima pubblicazione |
| 1.22 | Contratti guidati dal consumatore: versionare un evento di integrazione | Chi si rompe se cambiamo lo schema di `TodoCompleted`, e come lo scopriamo prima del deploy? | Prossima pubblicazione |
| 1.23 | Fitness function: l'architettura evolutiva si misura | Quali proprietà del sistema vogliamo difendere con un test automatico, oltre alle dipendenze? | Prossima pubblicazione |
| 1.24 | Topologie di team e carico cognitivo: Conway letto al contrario | I confini dei moduli seguono i team o li costringono? | Prossima pubblicazione |
| 1.25 | Multi-tenancy: schema condiviso, schema per tenant o database per tenant? | Quanto isolamento ci serve davvero, e quanto costa in migrazioni e operazioni? | Prossima pubblicazione |
| 1.26 | Feature flag come confine temporale | Come rilasciamo una regola nuova a metà degli utenti senza due versioni del dominio? | Prossima pubblicazione |
| 1.27 | Il costo del cloud come vincolo architetturale | Serverless, container o un'unica VM: quale forma di deploy paga meno per questo carico? | Prossima pubblicazione |

### Sezione 2: l'AI nelle pratiche di architettura

Il filo conduttore è quello del capitolo 1.7: il dominio non conosce HTTP, e non deve conoscere nemmeno il modello. Una sola funzione attraversa questi capitoli: l'utente incolla gli appunti di una riunione e il modello propone i todo da creare.

| # | Capitolo | Domanda centrale | Stato |
| --- | --- | --- | --- |
| 2.1 | [Il dominio non dovrebbe conoscere il modello](articles/2.01-domain-llm-boundary.md) | Se l'LLM sparisse domani, il caso d'uso avrebbe ancora senso? Dove vive l'adapter? | Bozza |
| 2.2 | [Una chiamata al modello è una chiamata remota](articles/2.02-llm-call-is-remote-call.md) | Timeout, retry, idempotenza e costi in token: quale parte della resilienza già nota si applica? | Bozza |
| 2.3 | [L'output del modello è input non fidato](articles/2.03-llm-output-untrusted-input.md) | Validiamo la risposta del modello come validiamo una richiesta HTTP? Dove fermiamo una prompt injection? | Bozza |
| 2.4 | Esporre i casi d'uso come tool: l'agente chiama l'application service, non il database | Se un agente crea un todo, chi fa rispettare il limite dei tre attivi? | Prossima pubblicazione |
| 2.5 | Serve davvero un vector database? | `pgvector` nel database che abbiamo o un servizio dedicato: quale complessità giustifica il secondo? | Prossima pubblicazione |
| 2.6 | Gli eval sono i test di un confine non deterministico | Come dimostriamo che una funzionalità basata su LLM non è peggiorata dopo un cambio di prompt o di modello? | Prossima pubblicazione |
| 2.7 | Prompt e versione del modello sono dipendenze da versionare | Cambiare prompt è un cambio di contratto? Come lo registriamo e come torniamo indietro? | Prossima pubblicazione |
| 2.8 | Human in the loop è un process manager | L'approvazione umana di un'azione proposta dall'agente è un passo di una saga: dove si salva lo stato? | Prossima pubblicazione |
| 2.9 | I test di architettura come guardrail per il codice generato | Come impediamo che un agente di coding eroda i confini più velocemente di quanto un umano li riveda? | Prossima pubblicazione |
| 2.10 | Il debito da comprensione: velocità generata, interessi pagati in revisione | Quanto codice possiamo accettare senza averlo capito, e chi ne paga gli interessi? | Prossima pubblicazione |
| 2.11 | ADR e documentazione come contesto per gli agenti | Le decisioni scritte per gli umani bastano a un agente, o il repository deve spiegare i propri confini? | Prossima pubblicazione |
| 2.12 | L'LLM come consumer di eventi di integrazione | Classificare i todo in modo asincrono a partire da `TodoCreated`: consistenza eventuale con un passo non deterministico | Prossima pubblicazione |

## Struttura del repository

```text
articles/
├── 1.01-business-before-architecture.md       Capitolo 1.1
├── 1.02-modular-monolith.md                   Capitolo 1.2
├── 1.03-cqrs-overkill.md                      Capitolo 1.3
├── 1.04-domain-vs-integration-events.md       Capitolo 1.4
├── 1.05-outbox-pattern.md                     Capitolo 1.5
├── 1.06-bounded-contexts.md                   Capitolo 1.6
├── 1.07-domain-http-error-mapping.md          Capitolo 1.7
├── 1.08-architecture-boundary-tests.md        Capitolo 1.8
├── 1.09-transactions-eventual-consistency.md  Capitolo 1.9
├── 1.10-distributed-systems-cost.md           Capitolo 1.10
├── 1.11-from-big-ball-of-mud.md               Capitolo 1.11
├── 1.12-technical-debt.md                     Capitolo 1.12
├── 2.01-domain-llm-boundary.md                Capitolo 2.1
├── 2.02-llm-call-is-remote-call.md            Capitolo 2.2
└── 2.03-llm-output-untrusted-input.md         Capitolo 2.3
examples/                                      Riservata a piccoli esempi autonomi
diagrams/                                      Illustrazioni SVG richiamate negli articoli
```

Nella prima sezione, i capitoli da 1.1 a 1.10 seguono un percorso dal problema di business ai costi operativi delle scelte architetturali; il capitolo 1.11 applica quel percorso a un sistema esistente e il capitolo 1.12 spiega come evitare di tornarci. La seconda sezione introduce un modello linguistico nell'applicazione Todo e mostra che le decisioni restano le stesse: confini espliciti, politiche per le chiamate remote e validazione di ciò che entra. Codice e diagrammi accompagnano il testo quando aiutano a chiarire una specifica decisione.

# Una chiamata al modello è una chiamata remota

La proposta di todo dagli appunti è in produzione da un mese. Un lunedì mattina il fornitore rallenta: le richieste di suggerimento restano appese per venti secondi, il pool di connessioni in uscita si riempie e anche la creazione manuale dei todo, che con il modello non ha nulla a che fare, comincia a restituire errori. Nello stesso mese la fattura del fornitore è triplicata: un utente ha incollato un documento di cento pagine e il frontend, non ricevendo risposta, ha riprovato tre volte.

Il [decimo capitolo](10-distributed-systems-cost.md#una-chiamata-di-rete-richiede-una-politica) dice che una chiamata di rete richiede una politica: timeout, tentativi, cosa fare quando la dipendenza non risponde. Il modello è una dipendenza di rete come lo sarebbe Account estratto in un servizio, con tre differenze che cambiano la politica: risponde in secondi invece che in millisecondi, costa in proporzione a quanto gli mandiamo e a quanto ci risponde, e a parità di richiesta non restituisce la stessa risposta.

**Timeout, retry, idempotenza e costo: quale parte di ciò che sappiamo già si applica, e cosa cambia?**

## Il problema: una dipendenza lenta e a pagamento sul percorso dell'utente

Nel monolite modulare una creazione di todo attraversa una transazione PostgreSQL che dura millisecondi. La chiamata al modello dura secondi, con una varianza alta e code nei picchi di carico del fornitore. Mentre aspetta, il caso d'uso occupa una connessione HTTP in ingresso, una in uscita e, se qualcuno ha aperto la transazione prima della chiamata, anche una connessione al database e i lock che ha preso.

L'ultimo caso è il più costoso e il più facile da scrivere per sbaglio: leggere il conteggio dei todo attivi con `FOR UPDATE`, chiamare il modello, poi creare. Il lock per utente del [monolite modulare](02-modular-monolith.md#dove-vive-la-regola-dei-tre-todo-attivi) resterebbe preso per tutta la durata della generazione. La [separazione del capitolo precedente](13-domain-llm-boundary.md) evita il problema per costruzione: `ProposeTodos` non apre transazioni, e `CreateTodo` non chiama il modello.

I guasti hanno forme diverse e richiedono risposte diverse:

| Esito della chiamata | Cosa sa il chiamante | Risposta ragionevole |
| --- | --- | --- |
| Timeout | Esito sconosciuto, ma nessun effetto sul nostro stato | Riprovare è sicuro per i dati, non per il costo |
| Limite di richieste superato (`429`) | Il fornitore chiede di rallentare | Attendere con backoff; riprovare subito peggiora |
| Errore del fornitore (`5xx`) | Guasto temporaneo dall'altra parte | Un tentativo in più, se il tempo lo consente |
| Risposta non valida | La chiamata è riuscita, il contenuto no | Al massimo un tentativo; poi rinunciare |
| Credenziali o configurazione errate | Riprovare non cambia nulla | Fallire subito e allarmare |
| Contenuto rifiutato dal fornitore | Esito definitivo per questo input | Non riprovare; spiegare all'utente |

La prima riga è la differenza principale rispetto ad Account. Un timeout verso Account lascia il dubbio se il comando sia stato eseguito. Una chiamata di sola generazione non modifica i nostri dati: ripeterla non duplica nulla. Costa però una seconda volta, e restituisce una risposta diversa. L'idempotenza qui riguarda il costo e la coerenza di ciò che l'utente vede, non la consistenza del database. Il discorso cambia il giorno in cui il modello riceve strumenti con effetti: a quel punto un timeout torna a essere un esito sconosciuto, come per Account, e ogni strumento ha bisogno del proprio identificativo idempotente.

## La soluzione più semplice: un limite all'ingresso, un timeout, una via d'uscita

Tre decisioni coprono la maggior parte dei casi del percorso interattivo.

Un limite alla dimensione dell'input, applicato al confine HTTP prima di chiamare il modello. Il costo di una chiamata cresce con la lunghezza degli appunti; un limite di caratteri concordato con il prodotto è anche un limite di spesa per richiesta. Un documento di cento pagine viene rifiutato con un messaggio, non pagato.

Un timeout derivato dal tempo che l'interfaccia è disposta ad aspettare, non dal tempo che il fornitore impiega. Se il prodotto accetta pochi secondi di attesa per un suggerimento, quello è il budget, e comprende l'eventuale tentativo ripetuto. Un timeout più lungo del budget dell'interfaccia non salva la risposta: l'utente ha già chiuso la scheda e noi stiamo pagando una generazione che nessuno leggerà.

Una via d'uscita quando il modello non risponde. Poiché il dominio non dipende dal modello, il caso d'uso può restituire una lista vuota e l'interfaccia mostra il modulo di creazione manuale. La funzione degrada invece di propagare il guasto. Questa possibilità è il beneficio concreto della separazione del capitolo precedente: senza di essa, un fornitore lento diventa un sistema lento.

I tentativi ripetuti stanno in un solo posto, l'adapter, con attesa crescente e una componente casuale, solo per `429`, `5xx` e timeout, e solo se il tempo residuo del budget lo permette. Il frontend non riprova per conto suo: se lo fa, ogni guasto del fornitore moltiplica le richieste proprio quando il fornitore è in difficoltà, come avverte il decimo capitolo per i retry a più livelli.

```ts
// todo/infrastructure/llm/provider-todo-suggester.ts (estratto)
async suggest(input: SuggestInput, deadline: Deadline): Promise<string[]> {
  for (let attempt = 1; ; attempt++) {
    try {
      const response = await this.client.generate(this.request(input), {
        timeoutMs: deadline.remainingMs(),
      });
      return parseSuggestion(response.text);
    } catch (error) {
      const wait = backoffMs(attempt);
      const outOfTime = deadline.remainingMs() < wait + MIN_CALL_MS;
      if (!isRetryable(error) || attempt >= 2 || outOfTime) {
        throw new SuggestionUnavailable(error);
      }
      await sleep(wait);
    }
  }
}
```

`SuggestionUnavailable` è un errore tecnico, non di dominio. Il caso d'uso lo traduce in una lista vuota e lo registra; non diventa un `409`, per la stessa ragione per cui nel [settimo capitolo](07-domain-http-error-mapping.md) un database irraggiungibile non diventa un falso «limite raggiunto».

## Isolare la dipendenza lenta dal resto

Il lunedì del fornitore lento ha fermato anche la creazione manuale perché le chiamate al modello condividevano con tutto il resto le connessioni in uscita e i worker del server. Il rimedio è un limite di concorrenza dedicato: un numero massimo di chiamate al modello in corso contemporaneamente, oltre il quale la richiesta di suggerimento fallisce subito con la via d'uscita, senza mettersi in coda. È il principio delle paratie di una nave: il compartimento del modello può allagarsi senza che affondi il resto.

Un interruttore completa il quadro: dopo una serie di fallimenti consecutivi l'adapter smette di chiamare il fornitore per un intervallo e restituisce subito `SuggestionUnavailable`, lasciando passare una chiamata di tanto in tanto per capire se il servizio è tornato. Questo evita di pagare timeout a raffica durante un'interruzione nota. Soglia e intervallo sono parametri da osservare, non da indovinare.

Una cache per richieste identiche copre un caso banale ma frequente: il doppio clic, il ricaricamento della pagina, il frontend che riprova nonostante tutto. Stessi appunti, stesso utente, stesso `maxItems` entro pochi minuti: stessa risposta, senza una seconda chiamata. Una cache semantica, che riconosca appunti simili, è un altro componente con i propri costi; non la introdurrei senza un dato che mostri richieste quasi uguali in quantità rilevante.

## Quando la chiamata esce dalla richiesta

Il prodotto chiede poi di classificare per progetto i todo già esistenti: decine di migliaia, una chiamata ciascuno. Non è lavoro per una richiesta HTTP. È lavoro per una tabella di job e un worker, con la stessa struttura dell'[outbox](05-outbox-pattern.md): ogni riga ha uno stato, un contatore di tentativi e una prenotazione con scadenza; il worker prende un lotto, chiama il modello, scrive il risultato e marca la riga.

```text
todo_classification_jobs
  todo_id, status (pending | done | failed), attempts, leased_until, result, last_error
```

Le responsabilità operative sono le stesse elencate per l'outbox: tentativi distanziati, quarantena per i job che falliscono sempre, metriche sul numero di pendenti e sull'età del più vecchio, pulizia. Se ne aggiunge una: il parallelismo del worker non si decide dalla CPU disponibile ma dal limite di richieste del fornitore. Dieci worker contro un limite di cinque richieste al secondo producono `429`, non velocità.

L'utente vede «classificazione in corso» e un risultato che arriva in ritardo. È un confine di consistenza eventuale come quelli di [Dove dovrebbe finire una transazione?](09-transactions-eventual-consistency.md), con un passo non deterministico nel mezzo.

## Osservare il costo, non solo gli errori

Per una dipendenza di rete si osservano latenza ed errori. Per il modello si aggiunge la spesa: token in ingresso e in uscita per chiamata, costo per funzione e per giorno, percentuale di risposte servite dalla cache, chiamate rifiutate dal limite di concorrenza e dall'interruttore. Un allarme sul costo giornaliero avrebbe segnalato il documento di cento pagine prima della fattura.

Correliamo i nostri log con l'identificativo di richiesta del fornitore, per aprire un ticket con qualcosa in mano. Non registriamo il testo completo degli appunti nei log: contiene dati personali, e la retention dei log non è quella concordata per quei dati.

## I compromessi e la decisione

La politica costa: un limite all'ingresso, un budget di tempo, un adapter con tentativi e interruttore, un limite di concorrenza, una cache, metriche sulla spesa. Sono componenti in più da configurare e da osservare. In cambio il fornitore lento non ferma la creazione dei todo, la spesa ha un tetto per richiesta e per giorno, e un'interruzione del servizio si manifesta come assenza di suggerimenti, non come indisponibilità dell'applicazione.

Per Todo scegliamo, nel percorso interattivo, un limite di caratteri sugli appunti, un budget di pochi secondi concordato con il prodotto, al massimo un tentativo ripetuto dentro quel budget, la lista vuota come via d'uscita, un limite di concorrenza dedicato e una cache per richieste identiche. Per la classificazione in massa, una tabella di job con un worker il cui parallelismo segue il limite del fornitore.

Verifichiamo tre comportamenti: con il fornitore che non risponde, la creazione manuale continua a funzionare e il suggerimento fallisce entro il budget; con il fornitore che restituisce `429`, un solo tentativo ripetuto e poi la via d'uscita; con un input oltre il limite, nessuna chiamata e nessun costo. Sono i test che rendono utile la politica.

**Il modello è una dipendenza remota, lenta e a pagamento.** Timeout, tentativi e isolamento sono gli stessi di qualsiasi chiamata di rete; cambia che ogni tentativo ha un prezzo e che la risposta non è mai due volte la stessa.

---

[Capitolo precedente: Il dominio non dovrebbe conoscere il modello](13-domain-llm-boundary.md)

[Capitolo successivo: L'output del modello è input non fidato](15-llm-output-untrusted-input.md)

[Torna all'indice dei capitoli](../README.md#capitoli)

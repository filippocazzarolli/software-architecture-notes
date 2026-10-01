# Il dominio non dovrebbe conoscere il modello

Il prodotto chiede una funzione nuova: l'utente incolla gli appunti di una riunione e l'applicazione propone i todo da creare. La prima versione è pronta in un pomeriggio. Il servizio che gestisce i todo importa l'SDK del fornitore, compone il prompt, legge la risposta e salva le attività. Due settimane dopo il fornitore cambia il formato della risposta, i test del limite dei tre attivi falliscono in CI perché manca la chiave API, e nessuno sa dire se il modello possa creare un quarto todo attivo.

Il [settimo capitolo](07-domain-http-error-mapping.md) ha separato il dominio da HTTP con una domanda: il dominio avrebbe ancora senso senza il protocollo? La stessa domanda vale per un modello linguistico.

**Se il modello sparisse domani, il caso d'uso avrebbe ancora senso? E dove vive l'adapter?**

## Il problema: una dipendenza che decide al posto del dominio

La versione del pomeriggio somiglia a questa:

```ts
// todo/domain/todo-suggestions.ts
import { ProviderClient } from "@provider/sdk";

export async function suggestTodos(notes: string, activeCount: number) {
  const client = new ProviderClient();
  const response = await client.generate({
    prompt: `Proponi al massimo ${3 - activeCount} todo da questi appunti: ${notes}`,
  });
  return JSON.parse(response.text) as { title: string }[];
}
```

Quattro problemi convivono in dieci righe.

L'import dell'SDK nel dominio è la stessa dipendenza di `@nestjs/common` nel settimo capitolo, con un'aggravante: il pacchetto cambia più spesso, e cambia anche il comportamento a parità di codice, perché il fornitore aggiorna il modello sotto lo stesso nome.

La regola dei tre todo attivi è finita in una frase del prompt. Un prompt è una richiesta, non un vincolo: il modello può restituire quattro titoli, o tre quando il conteggio nel frattempo è cambiato. La regola che il [monolite modulare](02-modular-monolith.md#dove-vive-la-regola-dei-tre-todo-attivi) protegge con una transazione viene qui affidata a un testo.

I test del dominio non sono più deterministici né eseguibili senza rete. Un test che chiama il modello verifica il fornitore, non la regola.

`JSON.parse` sulla risposta la tratta come fidata. È l'argomento del [capitolo sull'output del modello](15-llm-output-untrusted-input.md).

## La soluzione più semplice: stabilire cosa decide il dominio

Prima di scrivere l'adapter conviene separare due responsabilità. Il dominio decide cosa è un todo valido, con un titolo non vuoto e una lunghezza massima, e quanti ne può avere attivi un utente. Il modello trasforma un testo in titoli candidati. Dal punto di vista del dominio, un titolo proposto dal modello non è diverso da un titolo digitato dall'utente: è un input che entra nel caso d'uso di creazione e ne subisce le regole.

Questo chiarisce anche cosa non è il modello: non è un domain service. Un domain service esprime una regola del dominio in modo deterministico. Il modello produce candidati plausibili. Una proposta non è un todo finché un caso d'uso non la accetta.

La funzione diventa allora due casi d'uso. `ProposeTodos` chiede al modello dei candidati e li restituisce all'utente, senza scrivere nulla. `CreateTodo`, quello che esiste già, crea i todo confermati dall'utente con il controllo transazionale del limite. Il modello entra solo nel primo, attraverso un port:

```ts
// todo/application/ports/todo-suggester.ts
export interface TodoSuggester {
  suggest(input: { notes: string; maxItems: number }): Promise<string[]>;
}
```

Il port sta nel livello applicativo, non nel dominio: il dominio non ha bisogno di sapere che esiste un suggeritore. Restituisce stringhe, non entità, perché la trasformazione in `Todo` passa dal caso d'uso di creazione.

```ts
// todo/application/propose-todos.ts
import { MAX_ACTIVE_TODOS } from "../domain/active-todo-limit";
import type { TodoSuggester } from "./ports/todo-suggester";

export class ProposeTodos {
  constructor(
    private readonly suggester: TodoSuggester,
    private readonly todos: { countActive(ownerId: string): Promise<number> },
  ) {}

  async execute(command: { actorId: string; notes: string }): Promise<string[]> {
    const active = await this.todos.countActive(command.actorId);
    const room = MAX_ACTIVE_TODOS - active;
    if (room <= 0) return [];

    const titles = await this.suggester.suggest({ notes: command.notes, maxItems: room });
    return titles.slice(0, room);
  }
}
```

`maxItems` è un suggerimento al modello, utile per non pagare titoli che scarteremo. `slice` è la garanzia del caso d'uso. Il limite vero resta nella transazione di `CreateTodo`: fra la proposta e la conferma l'utente può aver attivato un altro todo da un'altra scheda, e la proposta non riserva alcun posto.

## Un esempio: l'adapter al confine

L'adapter vive in infrastructure e conosce tre cose che nessun altro deve conoscere: l'SDK, il prompt e il formato della risposta.

```ts
// todo/infrastructure/llm/provider-todo-suggester.ts
import { ProviderClient } from "@provider/sdk";
import type { TodoSuggester } from "../../application/ports/todo-suggester";
import { parseSuggestion } from "./parse-suggestion";

const SYSTEM_PROMPT = `Dal testo fornito estrai attività concrete da svolgere.
Rispondi solo con JSON nella forma { "titles": string[] }.`;

export class ProviderTodoSuggester implements TodoSuggester {
  constructor(
    private readonly client: ProviderClient,
    private readonly model: string,
  ) {}

  async suggest({ notes, maxItems }: { notes: string; maxItems: number }) {
    const response = await this.client.generate({
      model: this.model,
      system: SYSTEM_PROMPT,
      input: `Massimo ${maxItems} attività.\n\n<notes>${notes}</notes>`,
    });
    return parseSuggestion(response.text);
  }
}
```

I nomi dell'SDK sono illustrativi: ogni fornitore ha i propri, ed è esattamente il motivo per cui questa è l'unica classe che li conosce. `parseSuggestion` valida la risposta prima di restituirla, come un controller valida un corpo JSON; ne parla il [capitolo sull'output](15-llm-output-untrusted-input.md). L'identificativo del modello è un parametro di configurazione, non una costante nel codice: cambiarlo è un cambio di dipendenza da registrare, non un dettaglio.

Nei test del caso d'uso un `FakeTodoSuggester` restituisce titoli fissi. Possiamo verificare che con due todo attivi e cinque titoli proposti il risultato ne contenga uno, e che con tre attivi il modello non venga chiamato affatto. L'adapter ha i suoi test, pochi e separati, che parlano con il fornitore vero: girano su richiesta, non a ogni commit, perché costano e possono fallire per motivi che non riguardano il nostro codice.

Il confine si può proteggere come nel [capitolo sui test architetturali](08-architecture-boundary-tests.md): la regola che vieta al dominio ogni import esterno copre già l'SDK. Per il livello applicativo aggiungiamo una regola che rifiuta il pacchetto del fornitore, così il port resta l'unica via:

```js
{
  name: "application-no-llm-sdk",
  severity: "error",
  from: { path: "^src/[^/]+/application/" },
  to: { path: "^node_modules/@provider/" },
},
```

## Quando la separazione non serve ancora

Se la funzione è un esperimento per capire se gli utenti la vogliono, chiamare l'SDK direttamente dal servizio applicativo è un compromesso accettabile, con due condizioni. L'import non entra nelle entità o nelle funzioni che esprimono le regole. E il compromesso finisce nel [registro del debito](12-technical-debt.md#la-soluzione-più-semplice-scrivere-il-debito-quando-lo-si-contrae), con la condizione «prima del secondo uso del modello o del primo cambio di fornitore».

Il segnale che il momento è arrivato è lo stesso del settimo capitolo: un secondo ingresso. Qui l'ingresso è una seconda funzione che usa il modello, per esempio la classificazione dei todo esistenti, oppure la richiesta di confrontare due fornitori. Senza il port, il confronto richiede di toccare la logica applicativa; con il port, è un secondo adapter.

## I compromessi e la decisione

La separazione costa un'interfaccia, un adapter, un fake e un'indirezione in più. Il prompt, che decide la qualità delle proposte, vive lontano dal caso d'uso che ne dipende: va trattato come codice, con revisione e versione, e un prossimo capitolo lo considera come dipendenza da versionare.

In cambio i test del dominio restano deterministici, il fornitore si può cambiare toccando una classe, e il limite dei tre attivi ha una sola casa, la stessa per l'utente, per l'importazione batch e per il modello.

Per Todo scegliamo il port nel livello applicativo, l'adapter in infrastructure e un fake nei test. La regola di business non compare nel prompt se non come suggerimento di quantità. Un test verifica che il modello non venga interrogato quando non c'è posto; un altro che una proposta confermata con tre attivi riceva lo stesso `409` di una creazione manuale. Rivedremo la forma del port quando la seconda funzione mostrerà cosa hanno davvero in comune le due chiamate: non prima, per non progettare un gateway generico per un solo utilizzo.

**Il modello propone; il dominio decide.** Se domani il fornitore sparisse, `ProposeTodos` restituirebbe una lista vuota e `CreateTodo` continuerebbe a funzionare esattamente come oggi.

---

[Capitolo precedente: Il debito tecnico si paga con gli interessi](12-technical-debt.md)

[Capitolo successivo: Una chiamata al modello è una chiamata remota](14-llm-call-is-remote-call.md)

[Torna all'indice dei capitoli](../README.md#capitoli)

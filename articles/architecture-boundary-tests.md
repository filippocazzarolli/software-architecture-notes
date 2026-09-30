# Testare l'architettura, oltre alla logica di business

I test confermano che il quarto todo attivo viene rifiutato. Nel frattempo, per recuperare rapidamente un dato, qualcuno importa il repository di Account dentro Todo. Il comportamento rimane corretto e la CI passa, ma il confine concordato fra i moduli ha smesso di essere rispettato.

Un test del risultato non rileva necessariamente una dipendenza che renderà più costosa la prossima modifica.

**Come impediamo che i confini architetturali si erodano?**

## Il problema: le regole esistono soltanto nelle conversazioni

Abbiamo deciso che il dominio non conosce NestJS, che i casi d'uso non dipendono dai controller e che i contesti comunicano attraverso superfici pubbliche. Sono decisioni utili, ma una persona nuova nel team deve ricostruirle leggendo documenti, esempi e commenti nelle revisioni.

Sotto pressione, una dipendenza comoda sembra un'eccezione innocua. Dopo qualche mese altri file la copiano. La revisione architetturale diventa una discussione ripetitiva sugli stessi import, spesso quando il lavoro è già finito.

Automatizzare una parte di queste regole consente di ricevere un riscontro nel momento della modifica. Non dimostra che l'architettura sia buona: controlla che alcune decisioni esplicite continuino a valere.

## La soluzione più semplice: poche regole osservabili

Partirei dalle dipendenze che abbiamo già visto creare problemi, non da una matrice completa di tutti i file che possono comunicare.

| Regola | Modifica che vogliamo rendere sicura |
| --- | --- |
| Il dominio importa solo dal proprio dominio | Cambiare framework e persistenza senza riscrivere le regole |
| Application non importa presentation o infrastructure | Invocare i casi d'uso da ingressi diversi e sostituire gli adattatori |
| Gli altri contesti entrano solo da `public.ts` | Cambiare dettagli interni senza aggiornare tutti i chiamanti |
| Gli import devono essere risolvibili | Evitare che alias mal configurati nascondano dipendenze |

Nell'esempio adottiamo una politica volutamente restrittiva per il dominio, senza librerie esterne. Se una libreria pura diventasse utile, valuteremmo una deroga precisa. L'indipendenza non richiede per definizione di evitare ogni pacchetto.

Per pochi divieti può bastare una regola ESLint sugli import. Quando servono risoluzione degli alias e analisi del grafo, uno strumento dedicato evita di costruire un parser artigianale. Una ricerca testuale resta utile per esplorare, ma può ignorare riesportazioni, percorsi equivalenti e import multilinea.

## Un esempio: controllare le dipendenze risolte

Assumiamo questa struttura nel progetto applicativo:

```text
src/
├── account/
│   ├── public.ts
│   ├── domain/
│   ├── application/
│   └── infrastructure/
├── todo/
│   ├── public.ts
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
└── bootstrap.ts
```

Possiamo descrivere le dipendenze vietate con dependency-cruiser. Il [riferimento delle regole](https://github.com/sverweij/dependency-cruiser/blob/main/doc/rules-reference.md) documenta `from`, `to`, `pathNot` e i gruppi come `$1`, che permettono di confrontare il modulo di origine con quello di destinazione.

```js
// .dependency-cruiser.cjs — nella radice del progetto applicativo
module.exports = {
  forbidden: [
    {
      name: "domain-only-own-domain",
      severity: "error",
      from: { path: "^src/([^/]+)/domain/" },
      to: { pathNot: "^src/$1/domain/" },
    },
    {
      name: "application-no-adapters",
      severity: "error",
      from: { path: "^src/[^/]+/application/" },
      to: { path: "^src/[^/]+/(presentation|infrastructure)/" },
    },
    {
      name: "contexts-through-public-api",
      severity: "error",
      from: { path: "^src/(account|todo)/" },
      to: {
        path: "^src/(account|todo)/",
        pathNot: "^src/$1/|^src/(account|todo)/public\\.ts$",
      },
    },
    {
      name: "no-unresolved-imports",
      severity: "error",
      from: { path: "^src/" },
      to: { couldNotResolve: true },
    },
    {
      name: "no-circular-imports",
      severity: "error",
      from: { path: "^src/" },
      to: { circular: true },
    },
  ],
  options: {
    doNotFollow: { path: "node_modules" },
    tsConfig: { fileName: "tsconfig.json" },
    tsPreCompilationDeps: true,
  },
};
```

È un esempio da adattare, non una configurazione installata in questo repository di articoli. Nel progetto applicativo vanno installati dependency-cruiser e TypeScript come dipendenze di sviluppo, fissandone le versioni nel lockfile. Uno script npm può eseguire:

```sh
depcruise --config .dependency-cruiser.cjs --output-type err src
```

L'opzione `tsConfig` deve puntare alla configurazione che definisce gli alias effettivamente usati. `tsPreCompilationDeps` include le dipendenze TypeScript che non sopravvivono alla compilazione; `doNotFollow` evita di esplorare gli interni delle dipendenze installate senza nascondere gli import verso di esse. Questi comportamenti sono descritti nel [riferimento delle opzioni](https://github.com/sverweij/dependency-cruiser/blob/main/doc/options-reference.md).

La regola sui contesti copre qui soltanto Account e Todo: quando aggiungiamo Report dobbiamo estenderla. Il bootstrap è esterno ai contesti perché compone le implementazioni. Non deve diventare una directory in cui spostare logica per eludere i controlli.

![La CI risolve gli import, applica regole sulle dipendenze e segnala origine, destinazione e nome della regola violata. I test di comportamento restano distinti.](../diagrams/architecture-boundary-tests/dependency-check.svg)

*Figura 1 — Un errore utile indica la dipendenza da rimuovere e la decisione che protegge.*

## Verificare anche il controllo

Un comando verde può significare che non ci sono violazioni oppure che non è stato analizzato il codice giusto. Prepariamo piccole fixture che usino gli stessi alias e le stesse estensioni del progetto:

| Import nella fixture | Risultato atteso |
| --- | --- |
| `todo/application` → `todo/domain` | Consentito |
| `todo/infrastructure` → `account/public.ts` | Consentito |
| `todo/domain` → `@nestjs/common` | Errore |
| `todo/application` → `todo/presentation` | Errore |
| `todo/application` → `account/infrastructure` | Errore |
| Lo stesso accesso privato attraverso un alias o `import type` | Errore |

Eseguiamo lo strumento su ogni fixture e controlliamo sia l'esito sia il nome della regola. Una fixture che fallisce perché manca una dipendenza non dimostra che il confine venga rilevato correttamente. Almeno un caso deve essere valido, altrimenti un controllo che rifiuta tutto sembrerebbe funzionare.

Nel codice reale, una prova semplice consiste nell'introdurre temporaneamente un import vietato, osservare il fallimento e rimuoverlo. Ripeterei questa verifica quando cambia il resolver, il bundler o la struttura delle directory.

## Cosa il grafo non sa dire

Un `public.ts` può riesportare un repository interno. Gli altri moduli passano dal percorso consentito ma ricevono comunque dettagli privati. Il controllo degli archi diretti non valuta la qualità di quel contratto: servono revisione degli export o regole dedicate alle dichiarazioni pubbliche. Anche una dipendenza da `common/` può reintrodurre accoppiamento nascosto.

Gli import costruiti dinamicamente non sono sempre determinabili staticamente. Query SQL, permessi del database e chiamate HTTP verso altri servizi possono attraversare un confine senza produrre un import vietato. Per questi casi servono verifiche di integrazione e regole operative, come discusso nel [capitolo sui bounded context](bounded-contexts.md).

Infine, un grafo aciclico non garantisce il limite dei tre todo attivi. Quel comportamento richiede test della regola e prove concorrenti sul database. Il controllo architetturale risponde a una domanda diversa e si aggiunge agli altri test.

## Introdurre le regole in un progetto esistente

Se il progetto contiene molte violazioni, rendere subito bloccante ogni regola può fermare il lavoro senza migliorare il modello. Possiamo censire le dipendenze esistenti e vietarne di nuove, assegnando a ogni eccezione un responsabile e una condizione di rimozione.

Un'esclusione generica dell'intera directory `legacy/` tende però a diventare un rifugio permanente. Preferirei eccezioni circoscritte a dipendenze note e una riduzione progressiva. Una regola che tutti disabilitano segnala anche un possibile errore nella decisione architetturale: va discussa, non soltanto resa più severa.

## I compromessi e la decisione

Questi controlli riducono revisioni ripetitive e rendono osservabili le regressioni strutturali. Costano configurazione, tempo di CI e manutenzione quando cambia il progetto. Diventano rumore se impongono un'organizzazione senza una ragione concreta o se accumulano eccezioni incomprensibili.

Per Todo rendiamo bloccanti poche regole: indipendenza del dominio, casi d'uso senza adattatori, accessi pubblici tra contesti e import risolvibili. Controlliamo anche i cicli, verificando che i file pubblici non ne creino artificialmente. Manteniamo fixture che dimostrino il funzionamento del controllo.

La misura del successo è pratica: una scorciatoia che viola una decisione viene segnalata durante la modifica, con una spiegazione comprensibile. Il team può così correggerla oppure rivedere consapevolmente la decisione.

---

[Capitolo precedente: Il dominio non dovrebbe conoscere HTTP](domain-http-error-mapping.md)

[Capitolo successivo: Dove dovrebbe finire una transazione?](transactions-eventual-consistency.md)

[Torna all'indice dei capitoli](../README.md#capitoli)

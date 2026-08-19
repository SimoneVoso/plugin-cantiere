# claude-metodo

Il plugin **`cantiere`** per Claude Code: creare ed eseguire piani **a basso costo di contesto**, cioè
affrontare i lavori grossi senza che progettarli consumi tutto il contesto prima di cominciare.

Si chiama così perché è esattamente quello che gestisce: un cantiere aperto — un lavoro grande,
portato avanti a pezzi, dove ogni pezzo si chiude prima di aprire il successivo.

## L'idea, in tre righe

Una sessione **scrive** il piano e poi muore. Ogni capitolo del piano lo esegue una sessione
**nuova**, che riparte pulita e legge solo il capitolo suo. L'esplorazione — la voce di spesa più
grossa e la meno visibile — la fa un **agente di sola lettura su modello economico**, che legge
venti file e risponde con tre righe.

## Cosa contiene

| | Cosa fa |
|---|---|
| `/cantiere:piano <obiettivo>` | La sessione che progetta: intervista, esplora tramite l'agente, decide, e scrive un piano diviso in capitoli con la riga di stato in cima. Non esegue niente. |
| `/cantiere:passo <n>` | La sessione che esegue: legge solo il capitolo n, lo fa, lo verifica, committa, aggiorna lo stato e si ferma. |
| agente `lettore` | Sola lettura (`Read`, `Grep`, `Glob`) su modello `haiku`: trova dove stanno le cose e risponde con ancore `file:riga`. Non giudica e non può modificare niente. |

Il piano è **due file**: `PIANO_<NOME>.md`, con un tetto di 4 KB perché viene letto a ogni sessione,
e `PIANO_<NOME>_DOSSIER.md`, che non apre nessuno se non serve e dove quindi lo spazio è gratis.

### `lettore` non è solo per `/cantiere:piano` e `/cantiere:passo`

Le due skill lo chiamano sempre, ma **è un agente a sé**: una volta installato, resta disponibile
per qualunque domanda del tipo "dove sta / com'è fatto / esiste già", in qualunque momento della
sessione, anche senza aver lanciato nessuna delle due skill. Non serve chiederlo esplicitamente per
nome: la sua descrizione è scritta apposta perché Claude lo scelga da solo quando la domanda è di
quel genere — lo stesso meccanismo per cui sceglie l'agente `Explore` integrato, con la differenza
che `lettore` è pinnato su un modello economico e quello integrato di solito eredita il modello
principale.

**Non serve nemmeno installare il plugin per ottenere il risparmio**: qualunque subagent si lanci
con un parametro `model` esplicito (`haiku`, per esempio) legge al posto tuo a un costo più basso,
in ogni sessione, con o senza `cantiere`. Installare il plugin serve a **non doverlo chiedere ogni
volta**: la scelta diventa automatica, in ogni progetto dove `lettore` è disponibile.

## Installazione

Dentro Claude Code:

```
/plugin marketplace add SimoneVoso/claude-metodo
/plugin install cantiere@claude-metodo
```

Se il riepilogo dice `Run /reload-plugins to activate.`, lancia `/reload-plugins`.

**Scegli lo scope utente** ("install for yourself across all projects"), non quello di progetto: è
la differenza tra avere `lettore` in *questo* repository soltanto e averlo **in ogni progetto aperto
da questa macchina**, senza reinstallare né riconfigurare nulla per ciascuno.

Per farlo arrivare anche alle **sessioni cloud** (claude.ai/code), che girano su un container e non
vedono la tua `~/.claude` — nemmeno se hai installato a livello utente sul tuo PC — dichiaralo nel
`.claude/settings.json` **di ciascun repository** che userai da lì:

```json
{
  "extraKnownMarketplaces": {
    "claude-metodo": { "source": { "source": "github", "repo": "SimoneVoso/claude-metodo" } }
  },
  "enabledPlugins": ["cantiere@claude-metodo"]
}
```

## Perché una skill e non un `CLAUDE.md`

Di una skill resta in contesto **solo la descrizione** (~400 token); il corpo viene caricato quando
la skill è invocata, e i file di supporto solo se il corpo li apre. Le stesse trenta righe di metodo
scritte in un `CLAUDE.md` globale si pagherebbero in ogni sessione di ogni progetto, per sempre.

## Struttura

```
claude-metodo/
├── .claude-plugin/marketplace.json     il catalogo
└── plugins/cantiere/
    ├── .claude-plugin/plugin.json
    ├── agents/lettore.md               l'agente economico di sola lettura
    └── skills/
        ├── piano/SKILL.md + MODELLO.md il formato del piano
        └── passo/SKILL.md
```

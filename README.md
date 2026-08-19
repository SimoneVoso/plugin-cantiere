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

## Installazione

Dentro Claude Code:

```
/plugin marketplace add SimoneVoso/claude-metodo
/plugin install cantiere@claude-metodo
```

Se il riepilogo dice `Run /reload-plugins to activate.`, lancia `/reload-plugins`.

Per farlo arrivare anche alle **sessioni cloud** (claude.ai/code), che girano su un container e non
vedono la tua `~/.claude`, dichiaralo nel `.claude/settings.json` del progetto:

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

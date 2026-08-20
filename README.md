# plugin-cantiere

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

### Sistema a Fasi (UNICO)

| | Cosa fa |
|---|---|
| `/cantiere:apri-fasi <nome>` | Apri il cantiere: crea un documento `cantiere_<nome>.md` con tutte le fasi strutturate in italiano. Committa il file e si ferma. |
| `/cantiere:fase <n>` | Esegui una singola fase: legge il documento della fase, la esegue, verifica, committa, fai push automatico e crea/aggiorna la PR. Una fase per sessione. |

### Supporto

| | Cosa fa |
|---|---|
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
/plugin marketplace add SimoneVoso/plugin-cantiere
/plugin install cantiere@plugin-cantiere
```

Se il riepilogo dice `Run /reload-plugins to activate.`, lancia `/reload-plugins`.

**Scegli lo scope utente** ("install for yourself across all projects"), non quello di progetto: è
la differenza tra avere `lettore` in *questo* repository soltanto e averlo **in ogni progetto aperto
da questa macchina**, senza reinstallare né riconfigurare nulla per ciascuno.

Per le **sessioni cloud** (claude.ai/code) serve un passo in più. Girano su un container che non vede
la tua `~/.claude` — nemmeno se hai installato a livello utente sul tuo PC — e il
`.claude/settings.json` del repository **non basta da solo**: da scope di progetto Claude Code onora
`extraKnownMarketplaces`, e all'avvio clona e registra il marketplace, ma **non** `enabledPlugins`.
Dalla versione 2.1.195, un plugin che viene da una sorgente esterna e che solo il `settings.json` del
progetto abilita non si carica finché qualcuno non lo installa davvero.

Dichiarare il marketplace nel repository serve comunque, così nessuno deve aggiungerlo a mano:

```json
{
  "extraKnownMarketplaces": {
    "plugin-cantiere": { "source": { "source": "github", "repo": "SimoneVoso/plugin-cantiere" } }
  },
  "enabledPlugins": { "cantiere@plugin-cantiere": true }
}
```

`enabledPlugins` è un **oggetto**, non un elenco: la forma `["cantiere@plugin-cantiere"]` è quella
vecchia, e Claude Code chiede di migrarla. Nel cloud non installa niente in nessuna delle due forme,
ma vale per le sessioni locali e documenta l'intenzione.

L'installazione vera va nel **setup script dell'ambiente cloud**, che si configura su claude.ai e non
nel repository:

```bash
claude plugin marketplace add SimoneVoso/plugin-cantiere
claude plugin install cantiere@plugin-cantiere
```

Si scrive una volta sola e vale per ogni repository aperto da quell'ambiente. Se l'immagine del
container te la costruisci tu, l'alternativa è `CLAUDE_CODE_PLUGIN_SEED_DIR`, che pre-popola i plugin
a build time senza clonare niente all'avvio.

## Flusso di lavoro

Il sistema a fasi è l'**unico metodo** per gestire cantieri grandi. È pensato per strutture **flessibili**: puoi avere 3, 5, 10 fasi a seconda del cantiere.

### Come usare il sistema

1. **Apri il cantiere** (sessione 1):
   ```
   /cantiere:apri-fasi "Nome del cantiere"
   ```
   Crea il file `cantiere_<nome>.md` con tutte le fasi strutturate in italiano. Il file viene committato e pushato.

2. **Esegui ogni fase in una sessione nuova** (una sessione per fase):
   ```
   /cantiere:fase 1
   /cantiere:fase 2
   /cantiere:fase 3
   ...
   ```
   Ogni fase:
   - Legge il suo blocco da `fase_<nomeFase>_<nomeC antiere>.md`
   - Esegue il lavoro
   - Verifica il risultato
   - Committa, fai push e aggiorna la PR **automaticamente**
   - Quando conclusa, il file viene rinominato con `_conclusa`

3. **Chiudi il cantiere** (dopo l'ultima fase):
   - Fai merge della PR
   - Cancella il file `cantiere_<nome>.md` (non serve più)
   - Condensa le decisioni in CHANGELOG.md o docs/

### Vantaggi

- **Una sessione per fase**: ogni sessione reparte pulita, con memoria zero
- **Automatizzazione completa**: push e PR sono automatici, niente comandi manuali
- **Fasi flessibili**: puoi avere tante fasi quante ne servono
- **Tracciamento**: i file di fase si rinominano automaticamente quando conclusi
- **Italiano**: tutte le istruzioni e i messaggi sono in italiano
- **Verifiche obbligatorie**: ogni fase verifica che il lavoro sia fatto davvero

---

## Perché una skill e non un `CLAUDE.md`

Di una skill resta in contesto **solo la descrizione** (~400 token); il corpo viene caricato quando
la skill è invocata, e i file di supporto solo se il corpo li apre. Le stesse trenta righe di metodo
scritte in un `CLAUDE.md` globale si pagherebbero in ogni sessione di ogni progetto, per sempre.

## Struttura

```
plugin-cantiere/
├── .claude-plugin/marketplace.json        il catalogo
├── cantiere_<nome>.md                     documento principale del cantiere
├── fase_<nomeFase>_<nomeCantiere>.md     documenti delle fasi (rinominati _conclusa quando finiti)
└── plugins/cantiere/
    ├── .claude-plugin/plugin.json
    ├── agents/lettore.md                  l'agente economico di sola lettura
    └── skills/
        ├── apri-fasi/SKILL.md            apri cantiere con fasi
        └── fase/SKILL.md                 esecuzione fase singola + push/PR automatico
```

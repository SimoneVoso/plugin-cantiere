# plugin-cantiere

Il plugin **`cantiere`** per Claude Code: portare avanti lavori grossi **a basso costo di contesto**,
cioè senza che progettarli consumi tutto il contesto prima di cominciare.

Si chiama così perché è esattamente quello che gestisce: un cantiere aperto — un lavoro grande,
fatto a pezzi, dove ogni pezzo si chiude prima di aprire il successivo.

## L'idea, in tre righe

Una sessione **scrive** il piano e poi muore. Ogni fase del piano la esegue una sessione **nuova**,
che riparte pulita e legge solo la fase sua. L'esplorazione — la voce di spesa più grossa e la meno
visibile — la fa un **agente di sola lettura su modello economico**, che legge venti file e risponde
con tre righe.

## I comandi

| | Cosa fa |
|---|---|
| `/cantiere:piano <obiettivo>` | Apre il cantiere: scrive `PIANO.md` e un `FASE_<NN>_<slug>.md` per ogni fase, li committa e si ferma. **Non esegue niente.** Ha il **diritto di veto** su come si fa il piano. |
| `/cantiere:fase [n]` | **Un passo alla volta.** Esegue una fase sola, la verifica, la segna `✅ COMPLETA`, committa, pusha e apre o aggiorna la PR. Poi si ferma: la prossima si fa da una sessione nuova. |
| `/cantiere:veloce [n]` | **Cantiere veloce.** Fase → controllo → salva tutto → commit e push → ti consegna il `/clear` pronto per la fase dopo. Niente PR a ogni giro: quella la fa `/cantiere:chiudi`. |
| `/cantiere:notturno` | **Cantiere notturno.** Un giro all'ora: se una fase è in corso non fa nulla, altrimenti fa la prossima fase e la committa. Ogni **cinque fasi** consecutive inserisce una refattorizzazione, e alla fine passa alla fase conclusiva. Non fa domande. |
| `/cantiere:refattorizza [zona]` | Una passata di pulizia con le linee guida di **Robert C. Martin** (Clean Code, SOLID, regola del boy scout). Non cambia il comportamento, e i test devono essere verdi prima e dopo. |
| `/cantiere:chiudi` | La fase conclusiva: verifica che tutto giri davvero, condensa il perché nel diario del progetto, **cancella i file del cantiere** e lascia la PR pronta da fondere. |
| agente `lettore` | Sola lettura (`Read`, `Grep`, `Glob`) su modello `haiku`: trova dove stanno le cose e risponde con ancore `file:riga`. Non giudica e non può modificare niente. |

### Modello e impegno

Tutte le skill del cantiere girano su **`model: opus`** con **`effort: max`**, dichiarati nel
frontmatter: pianificare, verificare e decidere se una fase regge sono lavori in cui l'impegno si
ripaga. L'unica eccezione è voluta: l'agente `lettore` resta su **`haiku`**, perché il suo mestiere è
leggere tanto e costare poco — è il pezzo su cui si regge il risparmio di tutto il metodo.

## Il diritto di veto

Il cantiere non è un esecutore di ordini: **ha diritto di veto su come si fa il piano**, e le sue
nove regole stanno in [`plugins/cantiere/VETO.md`](plugins/cantiere/VETO.md).

In breve: niente fasi senza verifica eseguibile, niente fase che non stia in una sessione e in un
commit, ancore `file:riga` e non «nella zona di», nessuna fase che dipenda da scoperte di una fase
successiva, i nomi dei file sono quelli e non si negoziano, chi pianifica non esegue, una fase finita
si committa, niente allargamenti, un cantiere finito si smonta.

Quando la richiesta viola una regola, il cantiere **contesta e si ferma**:

```
VETO — regola 1: nessuna fase senza verifica eseguibile
Perché qui: la fase 2 dice «sistemare il login», e non si vede da cosa sia sistemato
Proposta: verifica «npm test -- auth» → 12 passed, e la schermata di login apre senza errori
```

Decide comunque l'utente: se rispondi di procedere lo stesso, si procede — e la forzatura si scrive
nella tabella *Veto e forzature* di `PIANO.md`, con il motivo. Un veto forzato e non scritto è un
veto che non è mai esistito.

## I nomi dei file, che sono sempre quelli

| File | Cosa contiene |
|---|---|
| `PIANO.md` | Radice del progetto. Riga di stato, decisioni, tabella delle fasi, forzature, scoperte. Tetto **4 KB**: viene letto a ogni sessione. |
| `FASE_<NN>_<slug>.md` | Una per fase, accanto al piano: `FASE_03_integrazione-github.md`. Obiettivo, file con ancore, cosa fare, verifica, trappole. |
| `PIANO_DOSSIER.md` | Facoltativo: il perché lungo, le alternative scartate. Nessun tetto — non lo apre nessuno se non serve. |
| `.cantiere/` | Fuori da git: il segnale «fase in corso» e il registro del cantiere notturno. |

Quando una fase finisce viene **segnata completa** in due posti, nello stesso commit del lavoro:
`**Stato: COMPLETA**` più lo sha nel file della fase, e `✅ **COMPLETA**` nella tabella di
`PIANO.md`. Il **nome del file non cambia mai**: rinominarlo a fase conclusa romperebbe i
riferimenti nel piano e nei commit già fatti.

E quando il cantiere è finito **e tutto gira davvero**, `/cantiere:chiudi` cancella i file nuovi —
`PIANO.md`, tutti i `FASE_*.md`, `.cantiere/` — dopo aver condensato il perché nel diario del
progetto. Se qualcosa è rosso non si cancella niente: il cantiere resta aperto, ed è la risposta
giusta.

## I tre modi di lavorare

Stesso piano, ritmi diversi. Si possono mescolare: sono tutti e tre lo stesso ciclo.

**Un passo alla volta** — quando il lavoro è delicato e ogni fase va rivista:

```
/cantiere:piano "migrazione a Postgres"     ← sessione 1: scrive il piano, si ferma
/cantiere:fase 1                            ← sessione 2: una fase, commit, push, PR
/cantiere:fase 2                            ← sessione 3: idem
```

**Veloce** — quando il piano è chiaro e vuoi macinare:

```
/cantiere:veloce      → fase, controllo, commit, push
/clear                → il contesto riparte vuoto
/cantiere:veloce      → fase successiva
```

**Notturno** — quando vuoi trovarti il lavoro fatto:

```
/loop 1h /cantiere:notturno
```

Un giro all'ora. Se una fase è in corso (segnale preso, copia di lavoro sporca, o un'altra sessione
avanti sul ramo) il giro **non fa nulla** e riprova l'ora dopo. Ogni cinque fasi consecutive fa una
passata di refattorizzazione alla Robert Martin, e quando le fasi finiscono passa alla fase
conclusiva. Non fa domande: se serve una decisione, la scrive in **Scoperte durante l'esecuzione**,
committa e si ferma. La mattina trovi un rapporto in poche righe.

Ogni giro finisce sempre con un commit **pushato**: il container di stanotte domattina non c'è più.

## Installazione

Dentro Claude Code:

```
/plugin marketplace add SimoneVoso/plugin-cantiere
/plugin install cantiere@plugin-cantiere
```

Se il riepilogo dice `Run /reload-plugins to activate.`, lancia `/reload-plugins`.

**Scegli lo scope utente** ("install for yourself across all projects"), non quello di progetto: è
la differenza tra avere il cantiere in *questo* repository soltanto e averlo **in ogni progetto
aperto da questa macchina**, senza reinstallare né riconfigurare nulla per ciascuno.

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

## `lettore` non è solo per il cantiere

Le skill lo chiamano sempre, ma **è un agente a sé**: una volta installato resta disponibile per
qualunque domanda del tipo «dove sta / com'è fatto / esiste già», in qualunque momento della
sessione, anche senza aver lanciato nessuna skill. Non serve chiederlo per nome: la sua descrizione è
scritta apposta perché Claude lo scelga da solo quando la domanda è di quel genere — lo stesso
meccanismo per cui sceglie l'agente `Explore` integrato, con la differenza che `lettore` è pinnato su
un modello economico e quello integrato di solito eredita il modello principale.

**Non serve nemmeno installare il plugin per ottenere il risparmio**: qualunque subagent lanciato con
un `model` esplicito (`haiku`, per esempio) legge al posto tuo a un costo più basso, in ogni
sessione. Installare il plugin serve a **non doverlo chiedere ogni volta**.

## Perché una skill e non un `CLAUDE.md`

Di una skill resta in contesto **solo la descrizione** (~400 token); il corpo viene caricato quando
la skill è invocata, e i file di supporto solo se il corpo li apre. Le stesse trenta righe di metodo
scritte in un `CLAUDE.md` globale si pagherebbero in ogni sessione di ogni progetto, per sempre.

## Struttura

```
plugin-cantiere/
├── .claude-plugin/marketplace.json     il catalogo
└── plugins/cantiere/
    ├── .claude-plugin/plugin.json
    ├── VETO.md                         le nove regole su cui il cantiere ha il veto
    ├── LOCK.md                         come si vede se una fase è già in corso
    ├── agents/lettore.md               l'agente economico di sola lettura
    └── skills/
        ├── piano/SKILL.md              apre il cantiere e scrive le fasi  (+ MODELLO.md)
        ├── fase/SKILL.md               una fase per sessione, con PR
        ├── veloce/SKILL.md             fase → controllo → commit → /clear
        ├── notturno/SKILL.md           un giro all'ora, da solo
        ├── refattorizza/SKILL.md       la passata alla Robert Martin
        └── chiudi/SKILL.md             verifica, condensa, cancella, PR pronta
```

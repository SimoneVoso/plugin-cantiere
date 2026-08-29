---
name: veloce
description: Cantiere veloce. Fa le fasi una dopo l'altra da solo - ogni fase la esegue l'agente esecutore in un contesto pulito, poi controlla che sia vero, e passa alla successiva senza fermarsi a farti premere niente. Prima di partire guarda quali fasi sono indipendenti e le manda insieme, in gruppi paralleli. Come /cantiere:fase, ma in catena e senza la PR a ogni giro.
argument-hint: "[numero della fase da cui partire]"
disable-model-invocation: true
model: sonnet
effort: medium
---

# /cantiere:veloce — le fasi una dopo l'altra, da sole

Fase di partenza: **$ARGUMENTS** (se è vuoto, la prima non `✅ COMPLETA` della tabella di
`PIANO.md`).

Qui **tu sei il caposquadra, non il muratore.** Non apri i file della fase, non scrivi codice, non
lanci `git diff`: ogni fase la esegue l'agente **`esecutore`** in un contesto suo, pulito, e ti
risponde con poche righe.

È esattamente per questo che le fasi possono andare **in catena**: il contesto sporco di una fase
non entra mai qui dentro, quindi non c'è niente da azzerare fra una fase e l'altra e non c'è nessun
`/clear` da far premere a nessuno. Se leggi tu i file della fase, la catena si rompe al terzo giro.

La PR non si apre a ogni fase: qui conta la velocità, e la PR la fa `/cantiere:chiudi` alla fine (o
`/cantiere:fase`, se ti serve subito). Se una PR è già aperta, il push la aggiorna da solo.

## Prima di cominciare

Cerca `PIANO.md` in radice. Se non c'è, di' di lanciare `/cantiere:piano` e fermati.

Segui `${CLAUDE_PLUGIN_ROOT}/LOCK.md`: se una fase risulta già in corso, **fermati e chiedi** — non
riaprire una fase aperta da qualcun altro.

### Il piano di marcia: cosa può andare in parallelo

**Questa è la prima cosa che fai, e la fai una volta sola**, prima del primo giro. Il piano è già
scritto: hai davanti tutte le fasi, i loro file e le loro dipendenze, ed è l'unico momento in cui
puoi vederle tutte insieme senza pagarle. Certe fasi non si toccano fra loro, e quelle possono
andare **insieme**: due o tre esecutori nello stesso giro, un commit solo.

Le condizioni stanno in `${CLAUDE_PLUGIN_ROOT}/PARALLELO.md` — leggilo adesso, sono quattro e vanno
verificate tutte. In breve: **file disgiunti, nessuna dipendenza, verifiche che convivono, fasi
normali**; nel dubbio, in fila.

Per deciderlo ti servono due cose, e nessuna delle due è aprire i file delle fasi:

1. La **tabella delle fasi** di `PIANO.md`, con la colonna *Dipende da*. La leggi comunque.
2. Le **sezioni `## File`** di ogni fase. Chiedile all'agente **`lettore`** con un incarico solo,
   per tutte le fasi in un colpo:

   > In questo repository, per ogni file `FASE_*.md` in radice, riporta il numero della fase, il
   > titolo, e l'elenco dei percorsi elencati nella sua sezione `## File`. Solo i percorsi, senza le
   > descrizioni. Aggiungi il comando della sezione `## Verifica`, in una riga.

Poi componi i gruppi come dice `PARALLELO.md` (dalla fase corrente in avanti, al massimo tre per
gruppo) e **annuncialo all'utente in poche righe**, prima di partire:

```
Piano di marcia — 7 fasi:
  fase 1 → da sola
  fasi 2 e 3 → insieme (file disgiunti: src/api/, src/ui/)
  fase 4 → da sola (tocca src/api/client.ts, come la 2)
  fasi 5, 6 e 7 → insieme
```

Un gruppo di una fase sola è il caso normale, non un fallimento: se il piano è una catena vera,
questo blocco ti costa un giro di `lettore` e l'analisi dice «tutte in fila». Va benissimo così,
dillo in una riga e parti.

**Il piano di marcia non è definitivo.** Se una fase riporta in **Scoperte** qualcosa che tocca i
file di una fase che avevi messo in gruppo con un'altra, il gruppo si scioglie: quelle fasi tornano
in fila. Si degrada verso la fila, mai il contrario.

## Il giro, che si ripete

Un giro è **una fase**, oppure **un gruppo parallelo**. Il resto del ciclo non cambia.

### 1. Guarda dove sei, e dillo

Leggi **solo** la riga di stato e la tabella delle fasi di `PIANO.md`. Qual è la prima fase non
`✅ COMPLETA`? Quella apre il giro; il piano di marcia dice se va da sola o in gruppo. Non aprire il
suo file: lo apre l'esecutore.

Poi, **prima di mandare chiunque**, scrivi all'utente la riga che dice a che punto siete:

```
**Fase 3 di 7** — Integrazione GitHub
```

e per un gruppo:

```
**Fasi 3 e 4 di 7**, in parallelo — Integrazione GitHub · Schermata impostazioni
```

Una riga, sempre, a ogni giro: è l'unica cosa che, mentre la catena macina, fa capire da fuori
quanto manca. Non saltarla perché «si vede dalla tabella»: la tabella non ce l'ha davanti nessuno.

### 2. Manda l'esecutore

Chiama l'agente **`esecutore`** con un incarico corto — non ha visto questa conversazione, ma legge
i file da solo, quindi non incollargli il piano:

> Esegui la fase **<NN>** del cantiere in questo repository. Leggi `PIANO.md` (riga di stato,
> Decisioni) e `FASE_<NN>_<slug>.md`, fai il lavoro, lancia la verifica, segna la fase completa,
> committa e pusha. Rispondi nella forma prevista.

**Se il giro è un gruppo parallelo**, le chiamate vanno **tutte nello stesso messaggio** — altrimenti
partono una dopo l'altra e non hai parallelizzato niente — e l'incarico cambia in due punti: dice
`in modalità parallela` e dice con chi:

> Esegui la fase **<NN>** del cantiere in questo repository, **in modalità parallela** (vedi il
> punto 0 delle tue istruzioni): insieme a te sta girando la fase **<MM>** nella stessa copia di
> lavoro. Leggi `PIANO.md` (riga di stato, Decisioni) e `FASE_<NN>_<slug>.md`, fai il lavoro, lancia
> la verifica, segna `COMPLETA` **solo nel file della tua fase**. Non toccare `PIANO.md`, non
> prendere il segnale, **non committare e non pushare**: al commit del gruppo ci penso io. Rispondi
> nella forma prevista, con i file toccati per intero.

Prima di mandarli, prendi tu il segnale per il gruppo (`${CLAUDE_PLUGIN_ROOT}/LOCK.md`, con
`fasi: 03,04`) e porta le righe del gruppo a `🔨 in corso`. È l'unica volta in cui il segnale lo
prende chi coordina, ed è perché gli esecutori sono più di uno e il segnale è uno solo.

### 3. Controlla che sia vero

Non ti fidi del rapporto: verificalo con quattro comandi, che costano poco e non ti riempiono.

```bash
git status --porcelain                      # vuoto: niente lasciato a metà
git log -1 --oneline                        # il commit della fase <NN>
git log @{u} -1 --oneline                   # lo stesso sha: è pushato davvero
grep -n 'FASE_<NN>' PIANO.md                # la riga dice ✅ COMPLETA
```

Poi **rilancia la verifica della fase** — il comando che l'esecutore ti ha riportato — e guarda
l'esito con i tuoi occhi. È l'unico controllo che vale il suo costo: una fase dichiarata fatta e mai
vista funzionare è il modo in cui un cantiere marcisce senza che nessuno se ne accorga.

**In un gruppo parallelo l'ordine è rovesciato**, perché il commit lo fai tu e viene per ultimo:

1. `git status --porcelain` **non** dev'essere vuoto: dev'essere sporco **esattamente** dei file che
   le fasi del gruppo hanno dichiarato (più i loro `FASE_*.md`). Un percorso che nessuna fase
   nominava è una collisione o un allargamento: fermati, non committare, vai al punto 5.
2. **Rilancia le verifiche una per volta**, non insieme, e guarda gli esiti.
3. Solo se è tutto verde: aggiorna `PIANO.md` (le righe del gruppo a `✅ **COMPLETA**` e la riga di
   stato in cima), poi il commit unico del gruppo e il push, come dice `PARALLELO.md`. Alla fine
   rilascia il segnale.

Se uno dei controlli non torna, o una verifica è rossa: **la fase non è chiusa.** Vai al punto 5.

### 4. Passa al giro dopo — senza chiedere

Il rapporto dell'esecutore stava in poche righe: tienile, buttane il resto, e **ricomincia dal punto
1** con la fase (o il gruppo) dopo. Non chiedere il permesso di continuare, non consegnare comandi da
premere, non proporre `/clear`: non serve, ed è tutto il senso di questa skill.

Non ti fermi perché sono passate tre fasi, e non ti fermi perché il cantiere è lungo. Ti fermi per i
motivi del punto 5, e per nessun altro.

Ogni tanto, fra un giro e l'altro, una riga sola all'utente: `Fase <NN> di <N> ✅ — <titolo>`.
Il racconto lungo lo farai alla fine.

### 5. Quando ti fermi

Quattro casi, e in tutti si scrive prima di fermarsi:

1. **Le fasi sono finite.** Tutte `✅ COMPLETA`: il cantiere è pronto da chiudere. Dillo **con i
   numeri** — «7 fasi su 7» — e **chiedi
   se chiuderlo** (`/cantiere:chiudi` verifica tutto, condensa il perché e cancella i file del
   cantiere). È l'unica domanda che questa skill fa: cancellare file e aprire la PR è una decisione,
   non una faccenda da sbrigare in automatico.
2. **La fase è rossa.** L'esecutore ha risposto `ROSSA`, o il tuo controllo del punto 3 non torna.
   Non rimandare lo stesso incarico una terza volta: scrivi cosa è successo in **Scoperte durante
   l'esecuzione** di `PIANO.md`, committa quella sola modifica, e fermati dicendo cos'è rosso. Se era
   un gruppo, salva prima la fase verde come dice `PARALLELO.md`: si committano i suoi file, non
   `-A`.
3. **Serve una decisione.** L'esecutore ha risposto `FERMATA`, o il veto è scattato su qualcosa che
   solo l'utente può forzare (`${CLAUDE_PLUGIN_ROOT}/VETO.md`). Porta la domanda all'utente con
   dentro quello che serve per rispondere, non la cronaca.
4. **Un'altra sessione sta lavorando.** Il segnale di `${CLAUDE_PLUGIN_ROOT}/LOCK.md` dice che una
   fase è in corso altrove: fermati e chiedi.

## Come chiudi

Poche righe, alla fine di tutta la catena. Si apre **sempre con i numeri** — `7 fasi su 7`, o
`5 su 7` se ti sei fermato prima, e in quel caso quali mancano: è la prima cosa che si guarda, e una
catena lunga senza quel conto costringe a contare le righe a mano. Poi: come si chiamavano le fasi,
quali sono andate in parallelo, cosa hanno stampato le verifiche in una riga l'una, cosa è finito in
**Scoperte**, e qual è il passo dopo. È l'unica cosa che l'utente leggerà davvero: mettici quello che
serve per decidere, non il diario dei giri.

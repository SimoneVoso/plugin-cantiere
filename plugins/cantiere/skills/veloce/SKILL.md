---
name: veloce
description: Cantiere veloce. Fa le fasi una dopo l'altra da solo - ogni fase la esegue l'agente esecutore in un contesto pulito, poi controlla che sia vero, e passa alla successiva senza fermarsi a farti premere niente. Come /cantiere:fase, ma in catena e senza la PR a ogni giro.
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
risponde con sei righe.

È esattamente per questo che le fasi possono andare **in catena**: il contesto sporco di una fase
non entra mai qui dentro, quindi non c'è niente da azzerare fra una fase e l'altra e non c'è nessun
`/clear` da far premere a nessuno. Se leggi tu i file della fase, la catena si rompe al terzo giro.

La PR non si apre a ogni fase: qui conta la velocità, e la PR la fa `/cantiere:chiudi` alla fine (o
`/cantiere:fase`, se ti serve subito). Se una PR è già aperta, il push la aggiorna da solo.

## Prima di cominciare

Cerca `PIANO.md` in radice. Se non c'è, di' di lanciare `/cantiere:piano` e fermati.

Segui `${CLAUDE_PLUGIN_ROOT}/LOCK.md`: se una fase risulta già in corso, **fermati e chiedi** — non
riaprire una fase aperta da qualcun altro. Il segnale della fase lo prende e lo rilascia
l'esecutore, non tu.

## Il giro, che si ripete

### 1. Guarda dove sei (poco, ogni volta)

Leggi **solo** la riga di stato e la tabella delle fasi di `PIANO.md`. Qual è la prima fase non
`✅ COMPLETA`? Quella è la fase del giro. Non aprire il suo file: lo apre l'esecutore.

### 2. Manda l'esecutore

Chiama l'agente **`esecutore`** con un incarico corto — non ha visto questa conversazione, ma legge
i file da solo, quindi non incollargli il piano:

> Esegui la fase **<NN>** del cantiere in questo repository. Leggi `PIANO.md` (riga di stato,
> Decisioni) e `FASE_<NN>_<slug>.md`, fai il lavoro, lancia la verifica, segna la fase completa,
> committa e pusha. Rispondi nella forma prevista.

Uno per volta: **non mandare due esecutori insieme**, nemmeno su fasi che sembrano indipendenti.
Due fasi in parallelo si pestano i piedi sul `PIANO.md` e producono un commit illeggibile (veto,
regola 2).

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

Se uno dei quattro non torna, o la verifica è rossa: **la fase non è chiusa.** Vai al punto 5.

### 4. Passa alla successiva — senza chiedere

Il rapporto dell'esecutore stava in sei righe: tienile, buttane il resto, e **ricomincia dal punto
1** con la fase dopo. Non chiedere il permesso di continuare, non consegnare comandi da premere, non
proporre `/clear`: non serve, ed è tutto il senso di questa skill.

Non ti fermi perché sono passate tre fasi, e non ti fermi perché il cantiere è lungo. Ti fermi per i
motivi del punto 5, e per nessun altro.

Ogni tanto, fra un giro e l'altro, una riga sola all'utente: `Fase <NN> ✅ — <titolo>`. Il racconto
lungo lo farai alla fine.

### 5. Quando ti fermi

Quattro casi, e in tutti si scrive prima di fermarsi:

1. **Le fasi sono finite.** Tutte `✅ COMPLETA`: il cantiere è pronto da chiudere. Dillo e **chiedi
   se chiuderlo** (`/cantiere:chiudi` verifica tutto, condensa il perché e cancella i file del
   cantiere). È l'unica domanda che questa skill fa: cancellare file e aprire la PR è una decisione,
   non una faccenda da sbrigare in automatico.
2. **La fase è rossa.** L'esecutore ha risposto `ROSSA`, o il tuo controllo del punto 3 non torna.
   Non rimandare lo stesso incarico una terza volta: scrivi cosa è successo in **Scoperte durante
   l'esecuzione** di `PIANO.md`, committa quella sola modifica, e fermati dicendo cos'è rosso.
3. **Serve una decisione.** L'esecutore ha risposto `FERMATA`, o il veto è scattato su qualcosa che
   solo l'utente può forzare (`${CLAUDE_PLUGIN_ROOT}/VETO.md`). Porta la domanda all'utente con
   dentro quello che serve per rispondere, non la cronaca.
4. **Un'altra sessione sta lavorando.** Il segnale di `${CLAUDE_PLUGIN_ROOT}/LOCK.md` dice che una
   fase è in corso altrove: fermati e chiedi.

## Come chiudi

Poche righe, alla fine di tutta la catena: quante fasi hai fatto e come si chiamavano, cosa hanno
stampato le verifiche in una riga l'una, cosa è finito in **Scoperte**, e qual è il passo dopo. È
l'unica cosa che l'utente leggerà davvero: mettici quello che serve per decidere, non il diario dei
giri.

---
name: fase
description: Esegue una sola fase del cantiere e si ferma. La fase la fa l'agente esecutore in un contesto pulito; qui si controlla che sia vero, si aggiorna e si apre o aggiorna la PR. Una fase per invocazione, per quando ogni passo va rivisto prima del successivo.
argument-hint: "[numero della fase]"
disable-model-invocation: true
model: sonnet
effort: medium
---

# /cantiere:fase — un passo alla volta

Fase richiesta: **$ARGUMENTS** (se è vuoto, quella indicata dalla riga di stato di `PIANO.md`).

Esegui **una fase sola**, poi ti fermi: non la successiva, nemmeno se avanza tempo o se sembra
piccola. Chi vuole macinare senza fermarsi ha `/cantiere:veloce`; questa skill esiste per il caso
opposto, quando ogni passo va guardato prima di aprire quello dopo.

E come là, **il lavoro non lo fai tu**: la fase la esegue l'agente **`esecutore`**, in un contesto
suo che non ha visto niente di questa conversazione. Tu leggi poco, controlli, e chiudi il giro con
la PR.

## 1. Leggi il minimo

Cerca `PIANO.md` in radice. Se non c'è, di' all'utente di lanciare `/cantiere:piano` e fermati.
Poi leggi **solo** la riga di stato in cima e la tabella delle fasi.

**Non aprire il file della fase**, e tantomeno quelli delle fasi successive: li apre l'esecutore, ed
è il motivo per cui questa sessione resta leggera.

Da quella tabella ricavi i due numeri che contano: **quale fase stai per fare e quante sono in
tutto**. Prima di mandare chiunque, scrivili all'utente, in una riga:

```
**Fase 3 di 7** — Integrazione GitHub
```

Vale sempre, anche quando le fasi sono due e sembra ovvio: è l'unica riga da cui, da fuori, si
capisce a che punto è il lavoro e quanto manca.

## 2. Controlla di poter partire

- Segui `${CLAUDE_PLUGIN_ROOT}/LOCK.md`: se una fase risulta in corso, **fermati e chiedi**. Il
  segnale lo prende e lo rilascia l'esecutore, non tu.
- La fase richiesta è quella che la riga di stato si aspetta? Se stai per rifarne una già
  `✅ COMPLETA`, o per saltarne una, dillo e chiedi conferma.

## 3. Manda l'esecutore

Chiama l'agente **`esecutore`** con un incarico corto — legge i file da solo, non incollargli il
piano:

> Esegui la fase **<NN>** del cantiere in questo repository. Leggi `PIANO.md` (riga di stato,
> Decisioni) e `FASE_<NN>_<slug>.md`, fai il lavoro, lancia la verifica, segna la fase completa,
> committa e pusha. Rispondi nella forma prevista.

Ti risponderà con poche righe: cosa ha fatto, cosa ha stampato la verifica, lo sha del commit, le
scoperte, la fase dopo. Il veto vale anche lì dentro (`${CLAUDE_PLUGIN_ROOT}/VETO.md`): se la fase
non ha una verifica eseguibile, o non sta in un commit solo, l'esecutore si ferma e te lo dice.

## 4. Controlla che sia vero

Non ti fidi del rapporto. Quattro comandi, che costano poco:

```bash
git status --porcelain                      # vuoto: niente lasciato a metà
git log -1 --oneline                        # il commit della fase <NN>
git log @{u} -1 --oneline                   # lo stesso sha: è pushato davvero
grep -n 'FASE_<NN>' PIANO.md                # la riga dice ✅ COMPLETA
```

Poi **rilancia la verifica della fase** e guarda l'esito con i tuoi occhi. Non dichiarare fatto ciò
che non hai visto funzionare, e non scrivere «dovrebbe funzionare».

Se qualcosa non torna, la fase **non è chiusa**: vai al punto 6.

## 5. Apri o aggiorna la PR

Il commit l'ha già fatto l'esecutore, con il lavoro e i due aggiornamenti di stato dentro. A te
resta la PR: aprila **se non esiste**, o aggiornane il corpo se c'è già. Una PR per cantiere, non
una per fase.

- **Titolo**: `Cantiere: <nome>`
- **Corpo**: la tabella delle fasi di `PIANO.md` con gli stati aggiornati, e sotto, per la fase
  appena chiusa, tre righe: cosa è stato fatto, cosa ha stampato la verifica, qual è la prossima.

## 6. Se la fase non regge, fermati e scrivilo

Se l'esecutore ha risposto `ROSSA` o `FERMATA`, o se il tuo controllo non torna, **non rimandarlo
una terza volta e non improvvisare un piano nuovo**: una fase corretta al volo, da chi ha appena
letto il rapporto, è esattamente come nascono i piani sbagliati. Scrivi cosa è successo in
**Scoperte durante l'esecuzione** di `PIANO.md`, committa quella sola modifica, e fermati dicendo
cosa serve decidere.

## 7. Chiudi il giro

Tre righe: cosa è stato fatto, cosa ha detto la verifica, qual è la fase successiva — e anche lì il
numero va sul totale: «prossima: fase 4 di 7». Poi:

> **Fase <NN> di <N> completata, committata e pushata.** La PR è aggiornata.
>
> Per la prossima: `/cantiere:fase <N+1>`, **anche da qui** — la fase l'ha eseguita l'esecutore nel
> suo contesto, questo è rimasto pulito e non c'è niente da azzerare.

Se era l'ultima fase, invece:

> **Tutte le fasi sono ✅ COMPLETE.**
>
> Lancia `/cantiere:chiudi`: verifica che tutto giri e smonta il cantiere (i file `PIANO.md` e
> `FASE_*.md` vanno cancellati, il perché va nel diario del progetto).

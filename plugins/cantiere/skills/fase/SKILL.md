---
name: fase
description: Esegue una sola fase del cantiere, la verifica, la segna COMPLETA, committa, fa il push e crea o aggiorna la PR, poi si ferma. Una fase per sessione, pronti a ripartire da una sessione nuova.
argument-hint: "[numero della fase]"
disable-model-invocation: true
model: opus
effort: max
---

# /cantiere:fase — un passo alla volta

Fase richiesta: **$ARGUMENTS** (se è vuoto, quella indicata dalla riga di stato di `PIANO.md`).

ultrathink

Esegui **una fase sola**. Non la successiva, nemmeno se avanza tempo o se sembra piccola: la
sessione dopo ripartirà pulita, ed è quello il punto del metodo.

## 1. Leggi il minimo

Cerca `PIANO.md` in radice. Se non c'è, di' all'utente di lanciare `/cantiere:piano` e fermati.
Poi leggi **solo**:

- la riga di stato in cima e la tabella delle fasi,
- la sezione **Decisioni**,
- il file `FASE_<NN>_<slug>.md` della fase richiesta, e nient'altro.

Non aprire i file delle fasi successive: non ti servono, e ciò che leggi lo paghi.

## 2. Controlla di poter partire

- Segui `${CLAUDE_PLUGIN_ROOT}/LOCK.md`: se una fase risulta in corso, **fermati e chiedi**.
- La fase richiesta è quella che la riga di stato si aspetta? Se stai per rifarne una già
  `✅ COMPLETA`, o per saltarne una, dillo e chiedi conferma.
- Il piano regge il veto (`${CLAUDE_PLUGIN_ROOT}/VETO.md`)? Se la fase che stai per eseguire non ha una verifica
  eseguibile, o non sta in un commit solo, non è una fase: contesta e fermati.
- Se il progetto ha documentazione propria (`CLAUDE.md`, `docs/`), leggila adesso.

Poi prendi il segnale e porta la fase a **in corso** (`${CLAUDE_PLUGIN_ROOT}/LOCK.md`).

## 3. Esegui, senza allargare

- Per ogni domanda «dove sta / com'è fatto / esiste già», chiama l'agente **`lettore`**: legge lui e
  ti risponde con ancore `file:riga`. Non aprire file per scoprire se sono quelli giusti.
- Apri e modifica **i file che la fase nomina**. Se te ne serve un altro, va bene; se te ne servono
  cinque, la fase era sbagliata: vedi il punto 7.
- Niente pulizie, rinomine o miglioramenti non chiesti (veto, regola 8). La refattorizzazione ha una
  fase sua: `/cantiere:refattorizza`.

## 4. Verifica davvero

Lancia la **Verifica** scritta nella fase e **guarda l'esito**. Se fallisce, la fase non è fatta:
correggi, rilancia, e non passare oltre finché non passa. Non dichiarare fatto ciò che non hai
visto funzionare, e non scrivere «dovrebbe funzionare».

## 5. Segna la fase COMPLETA

Due file, nello stesso commit del lavoro:

- in `FASE_<NN>_<slug>.md`: `**Stato: COMPLETA**` e lo sha del commit; compila la sezione **Fatto**
  con cosa hai fatto e cosa ha stampato la verifica;
- in `PIANO.md`: la riga della tabella diventa `✅ **COMPLETA**`, e la riga di stato in cima passa
  alla fase successiva:

```
**Fase corrente: <N+1> di <T>** · **Ultimo commit: <sha>** · **Stato: in esecuzione** · **Prossima azione: /cantiere:fase <N+1>**
```

Il nome del file della fase **non cambia**: lo stato sta dentro, non nel nome.

## 6. Committa, pusha, apri o aggiorna la PR

Un commit solo, con il lavoro e i due aggiornamenti di stato:

```bash
git add <file-modificati> PIANO.md FASE_<NN>_<slug>.md
git commit -m "cantiere fase <NN>: <titolo della fase>"
```

Se il progetto ha una convenzione sui commit o sul ramo, seguila. Poi il push, con quattro tentativi
e attese di 2s, 4s, 8s, 16s se la rete fa i capricci:

```bash
git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

Poi rilascia il segnale (`rm -f .cantiere/IN-CORSO`) e apri la PR **se non esiste**, o aggiornane il
corpo se c'è già. Una PR per cantiere, non una per fase.

- **Titolo**: `Cantiere: <nome>`
- **Corpo**: la tabella delle fasi di `PIANO.md` con gli stati aggiornati, e sotto, per la fase
  appena chiusa, tre righe: cosa è stato fatto, cosa ha stampato la verifica, qual è la prossima.

## 7. Se la fase non regge, fermati e scrivilo

Se scopri che la fase è incompleta, sbagliata o poggia su un presupposto falso, **non improvvisare
un piano nuovo**: una fase corretta a metà esecuzione, da chi ha il contesto pieno di codice, è
esattamente come nascono i piani sbagliati. Scrivi cosa hai trovato in **Scoperte durante
l'esecuzione** di `PIANO.md`, committa quella sola modifica, rilascia il segnale e fermati dicendo
cosa serve decidere.

## 8. Chiudi la sessione

Tre righe: cosa hai fatto, cosa ha detto la verifica, qual è la fase successiva. Poi:

> **Fase <NN> completata, committata e pushata.** La PR è aggiornata.
>
> **Apri una sessione nuova** e lancia `/cantiere:fase <N+1>`.

Se era l'ultima fase, invece:

> **Tutte le fasi sono ✅ COMPLETE.**
>
> **Apri una sessione nuova** e lancia `/cantiere:chiudi`: verifica che tutto giri e smonta il
> cantiere (i file `PIANO.md` e `FASE_*.md` vanno cancellati, il perché va nel diario del progetto).

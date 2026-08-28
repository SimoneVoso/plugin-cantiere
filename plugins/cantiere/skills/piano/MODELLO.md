# Il formato del cantiere

Due impalcature: una per `PIANO.md`, una per ogni `FASE_<NN>_<slug>.md`. Copiale e riempile; non
aggiungere sezioni che non servono. I nomi dei file sono quelli, sempre (veto, regola 5).

---

## `PIANO.md` — quello che si legge a ogni sessione (tetto: 4 KB)

```markdown
# Cantiere: <Nome> — piano

**Fase corrente: 1 di <N>** · **Ultimo commit: —** · **Stato: aperto** · **Prossima azione: /cantiere:fase 1**

<Due o tre righe: cosa deve essere vero alla fine del cantiere, e come si vedrà che lo è.>

## Decisioni

| # | Decisione | Perché |
|---|---|---|
| 1 | <cosa è stato deciso> | <una riga, non tre> |

## Fasi

| # | File | Titolo | Verifica | Stato |
|---|---|---|---|---|
| 1 | `FASE_01_<slug>.md` | <titolo> | `<comando>` | ⬜ da fare |
| 2 | `FASE_02_<slug>.md` | <titolo> | `<comando>` | ⬜ da fare |

Stati: ⬜ da fare · 🔨 in corso · ✅ **COMPLETA**

## Veto e forzature

| # | Regola forzata | Cosa è stato chiesto | Motivo dato |
|---|---|---|---|

<Vuota se il veto non è mai stato forzato. Se è vuota, si lascia vuota: non si cancella.>

## Scoperte durante l'esecuzione

<Vuoto all'inizio. Ci scrivono `/cantiere:fase` e `/cantiere:veloce` quando trovano qualcosa che il
piano non prevedeva.>
```

### Le righe che contano

- **La riga di stato**, in cima: permette di riprendere leggendo tre righe invece di ricostruire
  tutto. La aggiorna chi esegue, a ogni fase, nello stesso commit.
- **La tabella delle fasi** è l'indice del cantiere: una riga per fase, lo stato aggiornato lì e
  nel file della fase. Da lì si vede in un colpo d'occhio a che punto è il cantiere.
- **`Verifica`** dev'essere eseguibile da qualcun altro: `npm test -- auth` e cosa deve stampare,
  non «controllare che funzioni».

---

## `FASE_<NN>_<slug>.md` — una fase, una sessione (tetto: 4 KB)

````markdown
# Fase <NN> — <Titolo>

**Stato: DA FARE** · **Cantiere: <Nome>** · **Commit: —**

## Obiettivo

<Due righe: cosa deve essere vero alla fine di questa fase.>

## File

- `percorso/file.ext:120` — <cosa c'è lì>
- `altro/file.ext:34-41` — <cosa c'è lì>

## Cosa fare

1. <all'imperativo, un passo per riga>
2. <…>

## Verifica

```
<il comando da lanciare>
<cosa deve stampare perché la fase sia fatta>
```

## Trappole

- <una riga sola, e solo se la trappola esiste davvero>

## Fatto

<Vuoto all'inizio. Lo compila chi esegue: cosa ha fatto, cosa ha stampato la verifica.>
````

Quando la fase è finita, chi esegue cambia **due cose**, nello stesso commit del lavoro:

- qui, la riga di stato: `**Stato: COMPLETA**` e lo sha del commit;
- in `PIANO.md`, la riga della tabella: `✅ **COMPLETA**`, più la riga di stato in cima.

Il **nome del file non cambia mai**: rinominarlo a fase conclusa romperebbe i riferimenti in
`PIANO.md` e nei commit già fatti. Lo stato sta dentro il file, non nel nome.

---

## Dopo l'ultima fase

Un cantiere concluso **non resta un cantiere**. Lo smonta `/cantiere:chiudi`: verifica che tutto
giri, condensa il perché nel diario del progetto (`CHANGELOG.md`, `docs/`, quello che il progetto
usa) e **cancella i file nuovi** — `PIANO.md`, tutti i `FASE_*.md`, la cartella `.cantiere/`.

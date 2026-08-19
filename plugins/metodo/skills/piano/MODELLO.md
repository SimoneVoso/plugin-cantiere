# Il formato del piano

Due file. Copia le due impalcature qui sotto e riempile; non aggiungere sezioni che non servono.

---

## `PIANO_<NOME>.md` — quello che si legge a ogni sessione (tetto: 4 KB)

```markdown
# <Titolo del lavoro> — piano

**Capitolo corrente: 1 di <N>** · **Ultimo commit: —** · **Prossima azione: <una frase>**

<Due o tre righe: cosa deve essere vero alla fine, e come si vedrà che lo è.>

## Decisioni

| # | Decisione | Perché |
|---|---|---|
| 1 | <cosa è stato deciso> | <una riga, non tre> |

## Capitoli

### 1. <Titolo>
- **File**: `percorso/file.ext:120`, `altro/file.ext`
- **Cosa**: <due o tre righe, all'imperativo>
- **Verifica**: <il comando da lanciare e cosa deve stampare>
- **Trappola**: <una riga; il resto nel dossier §…>

### 2. <Titolo>
…

## Scoperte durante l'esecuzione

<Vuoto all'inizio. Ci scrive `/metodo:passo` quando trova qualcosa che il piano non prevedeva.>
```

### Le righe che contano

- **La riga di stato**, in cima: è ciò che permette di riprendere leggendo tre righe invece di
  ricostruire tutto. La aggiorna `/metodo:passo` a ogni capitolo, nello stesso commit.
- **`File:`** sono ancore vere, con il numero di riga dove il numero ha senso. È la differenza tra un
  capitolo eseguibile e un capitolo che va riesplorato.
- **`Verifica:`** dev'essere eseguibile da qualcun altro. "Controllare che funzioni" non è una
  verifica; `dotnet test` e cosa deve stampare, sì.
- **`Trappola:`** una riga sola, e solo se esiste davvero. Serve **prima** di sbagliare: la
  spiegazione lunga va nel dossier, l'avvertimento sta qui.

---

## `PIANO_<NOME>_DOSSIER.md` — quello che quasi nessuno apre

```markdown
# <Titolo del lavoro> — dossier

Il perché lungo del piano. **Non si legge per eseguire**: i capitoli citano solo i paragrafi che
servono.

## 1. <Titolo della decisione o della trappola>
<Tutto lo spazio che serve: le alternative, cosa è stato scartato e perché, il dettaglio tecnico,
i numeri, le prove.>

## 2. …
```

Qui dentro va tutto ciò che nel piano non entra: le alternative scartate (perché un giorno qualcuno
riproporrà quella scartata), i dettagli tecnici lunghi, i numeri delle verifiche fatte in fase di
studio, e le trappole spiegate per esteso.

---

## Dopo l'ultimo capitolo

Un piano concluso **non resta un piano**. Il perché — cioè la parte che vale ancora tra sei mesi —
si condensa nel diario delle decisioni del progetto (`STORICO.md`, `CHANGELOG.md`, quello che il
progetto usa), e i due file si cancellano. Un piano "Stato: fatto" lasciato in giro è peso che ogni
sessione futura si porterà dietro senza usarlo mai.

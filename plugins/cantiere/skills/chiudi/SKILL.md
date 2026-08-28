---
name: chiudi
description: La fase conclusiva del cantiere. Verifica che tutto giri davvero, condensa il perché nel diario del progetto, cancella i file del cantiere (PIANO.md, FASE_*.md, .cantiere/), committa, pusha e lascia la PR pronta da fondere.
disable-model-invocation: true
model: sonnet
effort: medium
---

# /cantiere:chiudi — smontare il cantiere

Un cantiere finito **non resta un cantiere**. I file del piano servivano a portare il lavoro da una
sessione all'altra: finito il lavoro sono peso morto, e un `PIANO.md` con «Stato: fatto» lasciato in
giro è peso che ogni sessione futura si porterà dietro senza usarlo mai (veto, regola 9).

Ma si cancella **solo se tutto gira**. L'ordine è: prima si verifica, poi si condensa, poi si
cancella. Mai al contrario.

## 1. Tutte le fasi sono chiuse?

Leggi la tabella di `PIANO.md`. Se una fase non è `✅ COMPLETA`, **fermati qui**: di' quale manca e
che va fatta (`/cantiere:fase <NN>`). Un cantiere non si chiude saltando una fase.

Se in **Scoperte durante l'esecuzione** c'è qualcosa di irrisolto, portalo all'utente adesso: è
l'ultimo momento in cui qualcuno lo leggerà.

## 2. Verifica che tutto giri, davvero

Non ti fidi delle spunte: quelle dicono che ogni fase funzionava **quando è stata fatta**, non che
funzionano tutte insieme adesso.

1. **Rilancia le verifiche di tutte le fasi**, una per una, e guarda gli esiti.
2. **Lancia i controlli del progetto** per intero: build, test, lint, typecheck — quello che il
   progetto ha.
3. **La copia di lavoro è pulita** e il ramo è pushato.
4. Se c'è una PR aperta con CI, **guarda che sia verde** sull'ultimo commit.

Se qualcosa è rosso: **non cancelli niente.** Correggi se è piccolo e dentro il perimetro del
cantiere, altrimenti fermati e di' cos'è rosso e perché. Il cantiere resta aperto: è la risposta
giusta, non un fallimento.

## 3. Condensa il perché

Il piano si cancella, il **perché** no: è la parte che vale ancora fra sei mesi. Prendi dalle
**Decisioni** di `PIANO.md` (e dalle forzature del veto, se ce ne sono state) quello che serve a chi
leggerà il codice senza aver visto il cantiere, e scrivilo dove il progetto tiene il suo diario:
`CHANGELOG.md`, `docs/`, `STORICO.md`, il `README.md`. Poche righe, al passato, senza la cronaca
delle sessioni.

Se il progetto non ha un posto dove metterlo, chiedi all'utente dove: non inventare un file nuovo
per l'occasione.

## 4. Cancella i file del cantiere

Adesso, e solo adesso:

```bash
git rm PIANO.md FASE_*.md
git rm -f PIANO_DOSSIER.md 2>/dev/null || true
rm -rf .cantiere
# togli anche la riga .cantiere/ dal .gitignore, se l'ha aggiunta il cantiere
```

Poi il commit finale, che contiene la cancellazione **e** le righe condensate nel diario:

```bash
git add -A
git commit -m "cantiere: chiuso — <nome>"
git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

## 5. Lascia la PR pronta

Apri la PR se non esiste, o aggiornala se c'è già. Titolo `Cantiere: <nome>`; nel corpo:

- **Cosa è cambiato**, in cinque righe, non fase per fase: il cantiere è finito, l'elenco dei passi
  non serve più a nessuno.
- **Le decisioni** che valgono ancora, e dove sono finite (il file del diario).
- **Come si verifica**: i comandi da lanciare e cosa devono stampare.
- **Le forzature del veto**, se ce ne sono state, con il motivo che era stato dato.

## 6. Chiudi

> **Cantiere chiuso.** ✅
>
> Tutte le fasi verificate, il perché è finito in `<file del diario>`, i file del cantiere
> (`PIANO.md`, `FASE_*.md`, `.cantiere/`) sono stati cancellati.
>
> La PR è pronta da rivedere e fondere: `<link>`.
>
> Per il prossimo lavoro: sessione nuova e `/cantiere:piano <obiettivo>`.

E se invece qualcosa era rosso, l'ultima riga è l'opposto, e va detta chiara:

> **Cantiere non chiuso**: `<cosa è rosso>`. I file del cantiere restano dove sono — si cancellano
> quando tutto gira, non prima.

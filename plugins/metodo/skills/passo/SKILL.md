---
name: passo
description: Esegue un solo capitolo di un piano scritto con /metodo:piano, lo verifica, lo committa e si ferma. Da lanciare in una sessione appena aperta, una per capitolo.
argument-hint: [numero del capitolo]
disable-model-invocation: true
---

# /passo — la sessione che esegue

Capitolo richiesto: **$ARGUMENTS** (se è vuoto, esegui quello indicato dalla riga di stato del piano).

Esegui **un capitolo solo**. Non il successivo, nemmeno se avanza tempo o se sembra piccolo: la
sessione dopo ripartirà pulita, ed è quello il punto del metodo.

## 1. Trova il piano e leggi il minimo

Cerca `PIANO_*.md` nel progetto (radice, `docs/`, la cartella dei piani). Se ce n'è più d'uno, chiedi
quale. Poi leggi **solo**:

- la riga di stato in cima,
- la sezione **Decisioni**,
- **il capitolo richiesto** e nient'altro.

**Non aprire il dossier**, a meno che il capitolo lo citi espressamente. Non leggere i capitoli
successivi: non ti servono, e ciò che leggi lo paghi.

## 2. Controlla di poter partire

- La copia di lavoro è pulita? Se ci sono modifiche non committate che non sono tue, chiedi prima di
  toccare qualcosa.
- Il capitolo richiesto è quello che la riga di stato si aspetta? Se stai per rifare un capitolo già
  fatto, o saltarne uno, **dillo e chiedi conferma**.
- Se il progetto ha una documentazione propria per la zona toccata, aprila adesso: è scritta per
  essere letta.

## 3. Esegui, senza allargare

- Per ogni domanda del tipo "dove sta / come è fatto / esiste già", chiama l'agente **`lettore`**:
  legge lui e ti restituisce ancore `file:riga`. Non aprire file per scoprire se sono quelli giusti.
- Apri e modifica **i file che il capitolo nomina**. Se ti accorgi che ne serve un altro, va bene —
  ma se ti accorgi che ne servono cinque, il capitolo era sbagliato: vedi il punto 6.
- Non fare pulizie, rinomine o miglioramenti non chiesti: allargare un capitolo è il modo più veloce
  di finire il contesto e di rendere il commit illeggibile.

## 4. Verifica davvero

Lancia la **Verifica** scritta nel capitolo e guarda l'esito. Se fallisce, il capitolo non è fatto:
correggi, oppure fermati e spiega. Non passare oltre con un "dovrebbe funzionare", e non dichiarare
fatto ciò che non hai visto funzionare.

## 5. Committa e aggiorna lo stato

Un commit solo, messaggio `piano <nome>: capitolo <n> — <titolo>`. Nello **stesso** commit aggiorna
la riga di stato del piano, e **solo quella riga**:

```
**Capitolo corrente: <n+1> di <N>** · **Ultimo commit: <sha>** · **Prossima azione: <una frase>**
```

Se il progetto ha una convenzione sul ramo o sulla firma dei commit, seguila.

## 6. Se il capitolo non regge, fermati e scrivilo

Se durante l'esecuzione scopri che il capitolo è incompleto, sbagliato o poggia su un presupposto
falso — **non improvvisare un piano nuovo**. Scrivi cosa hai trovato nella sezione **Scoperte
durante l'esecuzione** del piano, committa quella, e fermati dicendo cosa serve decidere. Un piano
corretto a metà esecuzione da chi ha il contesto pieno di codice è come nascono i piani sbagliati.

## 7. Chiudi

Riporta in tre righe: cosa è stato fatto, cosa ha detto la verifica, qual è il capitolo successivo.
Poi la frase che chiude ogni esecuzione:

> Capitolo <n> fatto e committato. **Apri una sessione nuova** e lancia `/metodo:passo <n+1>`.

Se era l'ultimo capitolo, invece: ricorda che un piano concluso non resta un piano — il perché va
condensato nel diario delle decisioni del progetto e i due file si cancellano.

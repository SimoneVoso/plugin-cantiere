---
name: fase
description: Esegui una singola fase del cantiere (0, 1, 2, o 3), verificala, committa il lavoro, fai il push e crea/aggiorna la PR automaticamente. Una fase per sessione, sempre.
argument-hint: [numero della fase 0-3]
disable-model-invocation: true
---

# /fase — Esegui una fase singola del cantiere

Fase richiesta: **$ARGUMENTS** (se vuoto, cerca il numero nella riga di stato di FASI.md)

Questa sessione esegue **una fase sola**. Non la successiva, nemmeno se avanza tempo. Ogni fase è una sessione nuova, completa, isolata.

## 1. Trova e leggi il documento FASI.md

Cerca `FASI.md` nella radice del progetto. Se non esiste, chiedi all'utente di lanciare `/cantiere:apri-fasi` prima.

Leggi **solo**:
- La riga di stato in cima
- La sezione della fase richiesta
- Nient'altro

Non leggere le fasi successive: non ti servono e ciò che leggi lo paghi in contesto.

## 2. Controlla di poter partire

- La copia di lavoro è pulita? Se ci sono modifiche non committate, chiedi conferma.
- La fase richiesta è quella che FASI.md si aspetta? Se stai per rifare una fase già fatta, **chiedi conferma**.
- Se il progetto ha documentazione propria (CLAUDE.md, docs/), leggila adesso.

## 3. Esegui la fase

- Per ogni domanda del tipo "dove sta / come è fatto", usa l'agente **`lettore`**: legge lui e ti restituisce ancore `file:riga`.
- Apri e modifica **solo i file che FASI.md nomina**.
- Non fare pulizie o miglioramenti non richiesti: allargare una fase è il modo più veloce di finire il contesto.

## 4. Verifica davvero

Esegui la verifica scritta in FASI.md e guarda l'esito. Se fallisce, la fase non è fatta:
- Correggi il codice
- Rilancia la verifica
- Non passare oltre finché non passa

Non dichiarare fatto ciò che non hai visto funzionare.

## 5. Committa il lavoro

Un commit solo con messaggio standardizzato:
```
cantiere fase <N>: <titolo della fase>
```

Esempio:
```
git add <file-modificati>
git commit -m "cantiere fase 1: implementazione principale"
```

Nello **stesso** commit, aggiorna la riga di stato di FASI.md:

```
**Fase corrente: <N+1> di 3** · **Ultimo commit: <sha>** · **Stato: In esecuzione**
```

Se il progetto ha una convenzione sui commit (firma, branch), seguila.

## 6. Push automatico

Fai il push del commit:

```bash
git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

Se il push fallisce per cause di rete, riprova fino a 4 volte con backoff esponenziale:
- Tentativo 1: subito
- Tentativo 2: attendi 2 secondi
- Tentativo 3: attendi 4 secondi
- Tentativo 4: attendi 8 secondi
- Tentativo 5: attendi 16 secondi

## 7. Crea o aggiorna la PR

Usa lo strumento GitHub per creare una PR (se non esiste) o aggiornarla (se già esiste):

**PR Title**: `Cantiere: fase <N> — <titolo>`

**PR Body**:
```markdown
## Cantiere: Fase <N>

### Cosa è stato fatto
<Descrizione breve in italiano di cosa è stato realizzato in questa fase>

### Verifica
La verifica della fase ha stampato:
\`\`\`
<output della verifica>
\`\`\`

### Note
- **Fase**: <N> di 3
- **Status**: Completata
- **Prossimo passo**: Lanciare `/cantiere:fase <N+1>` in una nuova sessione

---
_Cantiere: fase <N> completata_
```

Se la PR esiste già, aggiorna il body con il nuovo contenuto di questa fase.

## 8. Chiudi e riporta

Riporta in tre righe:
1. Cosa è stato fatto in questa fase
2. Esito della verifica
3. Prossima azione

Poi la frase che chiude:

> **Fase <N> completata e committata.** ✅
>
> Il commit è stato pushato e la PR è stata creata/aggiornata.
>
> **Apri una sessione nuova** e lancia `/cantiere:fase <N+1>` per continuare.

Se era l'ultima fase (fase 3), invece:

> **Cantiere chiuso!** 🎉
>
> Tutte le 4 fasi sono complete. Fai merge della PR, e cancella il file FASI.md (non serve più).
>
> Condensa le decisioni importanti nel CHANGELOG.md o nella documentazione del progetto.

## Cosa fare se la fase non regge

Se durante l'esecuzione scopri che la fase è incompleta o sbagliata:

1. Scrivi cosa hai trovato nella sezione **Scoperte durante l'esecuzione** di FASI.md
2. Committa quella sola modifica
3. Fermati e spiega cosa serve decidere

Non improvvisare un piano nuovo: il contesto è pieno di codice, le decisioni vanno prese con calma, non durante l'esecuzione.

## Convenzioni

- **Un commit per fase**: tutto il lavoro della fase va in un commit solo
- **Un push per commit**: dopo il commit, push subito
- **Una PR per cantiere**: la stessa PR si aggiorna a ogni fase, non ne crei una nuova
- **Una sessione per fase**: completa una fase e fermati, punto.

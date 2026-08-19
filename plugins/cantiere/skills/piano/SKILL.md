---
name: piano
description: Scrive un piano di lavoro diviso in capitoli, dove ogni capitolo è eseguibile da una sessione nuova con /cantiere:passo. Da usare quando un lavoro è troppo grosso per una sessione sola, o quando progettare rischia di consumare tutto il contesto.
argument-hint: [obiettivo del lavoro]
disable-model-invocation: true
---

# /piano — la sessione che progetta

Stai facendo **solo** il piano di: **$ARGUMENTS**

Questa sessione può consumare tutto il contesto che vuole: il suo unico prodotto è un file
committato. **Non esegui niente di ciò che pianifichi**, nemmeno il primo capitolo, nemmeno se
sembra banale: chi esegue è una sessione nuova, con `/cantiere:passo`.

## La regola che governa tutto il resto

**Ogni capitolo dev'essere eseguibile da una sessione appena nata**, che non ha visto questa
conversazione e non sa niente di ciò che hai scoperto. Tutto ciò che serve per eseguirlo sta scritto
nel capitolo; tutto ciò che non ci sta scritto è perso. È il metro con cui giudicare ogni riga che
scrivi: *"una sessione che legge solo questo, riesce a farlo senza riesplorare?"*

## Come procedi

### 1. Capisci l'obiettivo, e chiedi solo ciò che cambia il piano

Fai al massimo tre o quattro domande, e solo quelle le cui risposte **cambierebbero i capitoli**
(dove finisce il file, quale delle due strade, cosa succede ai dati esistenti). Usa
`AskUserQuestion`. Tutto ciò che puoi decidere da solo, decidilo e scrivilo tra le decisioni: una
domanda a cui esiste una risposta ovvia è contesto sprecato per entrambi.

### 2. Esplora **solo** tramite l'agente `lettore`

Non aprire file per scoprire se sono quelli giusti: è la voce di spesa più grossa e la meno visibile.
Chiedi al `lettore` (agente di sola lettura, modello economico) e ricevi ancore `file:riga`. Apri tu
un file solo quando sai già che è quello, e ti serve leggerlo davvero.

Se il progetto ha una documentazione propria (un `CLAUDE.md`, una cartella `docs/`), leggi **prima**
quella: è scritta per essere letta, il codice no.

### 3. Prendi le decisioni, e scrivile come decisioni

Ogni scelta fatta va nel piano con una riga di motivo. Servono a non rimetterle in discussione a
metà esecuzione: una decisione scritta si contesta con un motivo nuovo, una decisione ricordata si
rifà da capo ogni volta.

### 4. Dividi in capitoli

Un capitolo è **il lavoro di una sessione**, e si riconosce da tre cose:

- ha una **verifica eseguibile** alla fine (un comando, un test, una schermata da guardare) — se non
  sai dire come si vede che è fatto, non è un capitolo;
- sta in **un commit** che ha senso da solo;
- non dipende da cose scoperte in un capitolo successivo.

Meglio sei capitoli piccoli che tre grossi: un capitolo troppo grande è quello che farà finire il
contesto a metà, cioè esattamente il problema che questo metodo esiste per risolvere.

### 5. Scrivi i due file

Apri `MODELLO.md` (accanto a questo file) e segui quel formato. Sono due file:

| File | Cosa contiene | Tetto |
|---|---|---|
| `PIANO_<NOME>.md` | stato, decisioni in una riga l'una, capitoli con ancore e verifiche | **4 KB** |
| `PIANO_<NOME>_DOSSIER.md` | il perché lungo, le trappole spiegate, ciò che hai scartato e perché | nessuno |

Il tetto sul primo non è pignoleria: quel file viene letto **a ogni** sessione di esecuzione. Il
dossier non lo apre nessuno se non serve, quindi lì lo spazio è gratis. Se una cosa non entra nel
piano, non si accorcia: si sposta nel dossier e il capitolo la cita.

Scrivi i file **per sezioni**, non riscrivendoli da capo a ogni ripensamento: riscrivere un file
intero costa il suo peso ogni volta.

Chiedi all'utente dove metterli se il progetto ha una convenzione (una cartella `docs/`, una cartella
dei piani); altrimenti nella radice del progetto.

### 6. Committa il piano, e fermati

Commit del solo piano, messaggio `piano <nome>: scritto`. Poi **fermati** e di' all'utente, con
queste parole:

> Il piano è scritto e committato. **Apri una sessione nuova** e lancia `/cantiere:passo 1`.
> Questa sessione ha in pancia tutta l'esplorazione: eseguire da qui vanificherebbe il metodo.

## Le tre cose che rovinano un piano

1. **"Nella zona di…"** invece di `file.ext:120`. Chi esegue riesplora, e il metodo non serve più.
2. **La cronaca mescolata ai passi.** Il piano dice cosa fare; com'è andata si scrive dopo, altrove.
3. **Un capitolo senza verifica.** Diventa "fatto" perché nessuno ha guardato.

---
name: piano
description: Apre il cantiere. Scrive PIANO.md e un file FASE_<NN>_<slug>.md per ogni fase, con verifiche eseguibili e ancore file:riga, e li committa. Non esegue niente di ciò che pianifica. Ha diritto di veto su come si fa il piano.
argument-hint: "[obiettivo del cantiere]"
disable-model-invocation: true
model: opus
effort: max
---

# /cantiere:piano — la sessione che progetta

Stai facendo **solo** il piano di: **$ARGUMENTS**

ultrathink

Questa sessione può consumare tutto il contesto che vuole: il suo unico prodotto sono file
committati. **Non esegui niente di ciò che pianifichi**, nemmeno la prima fase, nemmeno se sembra
banale: chi esegue è una sessione nuova, con `/cantiere:fase 1`.

## 0. Il veto, prima di tutto

Apri `${CLAUDE_PLUGIN_ROOT}/VETO.md` (nella radice del plugin) e tienilo come metro di ogni riga che scrivi. Sono
nove regole, e su quelle il cantiere **ha diritto di veto**: se ciò che ti viene chiesto le viola,
contesti nella forma scritta lì e ti fermi. Non scrivi il piano nella forma vietata «intanto».

Il veto si forza solo con un sì esplicito dell'utente, e la forzatura finisce nella tabella *Veto e
forzature* di `PIANO.md`.

## La regola che governa tutto il resto

**Ogni fase dev'essere eseguibile da una sessione appena nata**, che non ha visto questa
conversazione e non sa niente di ciò che hai scoperto. Tutto ciò che serve per eseguirla sta scritto
nel file della fase; tutto ciò che non ci sta scritto è perso. È il metro di ogni riga:
*«una sessione che legge solo questo, riesce a farlo senza riesplorare?»*

## Come procedi

### 1. Capisci l'obiettivo, e chiedi solo ciò che cambia il piano

Al massimo tre o quattro domande, e solo quelle le cui risposte **cambierebbero le fasi** (dove
finisce il file, quale delle due strade, cosa succede ai dati esistenti). Usa `AskUserQuestion`.
Tutto ciò che puoi decidere da solo, decidilo e scrivilo tra le decisioni: una domanda che ha una
risposta ovvia è contesto sprecato per entrambi.

### 2. Esplora **solo** tramite l'agente `lettore`

Non aprire file per scoprire se sono quelli giusti: è la voce di spesa più grossa e la meno visibile.
Chiedi al `lettore` (sola lettura, modello economico) e ricevi ancore `file:riga`. Apri tu un file
solo quando sai già che è quello e ti serve leggerlo davvero.

Se il progetto ha una documentazione propria (`CLAUDE.md`, `docs/`), leggi **prima** quella: è
scritta per essere letta, il codice no.

### 3. Prendi le decisioni, e scrivile come decisioni

Ogni scelta fatta va nel piano con una riga di motivo. Una decisione scritta si contesta con un
motivo nuovo; una decisione ricordata si rifà da capo a ogni sessione.

### 4. Dividi in fasi

Una fase è **il lavoro di una sessione**, e si riconosce da tre cose: ha una verifica eseguibile
alla fine, sta in un commit che ha senso da solo, non dipende da cose scoperte in una fase
successiva. Sono le regole 1, 2 e 4 del veto, e valgono anche contro te stesso.

Meglio sei fasi piccole che tre grosse: una fase troppo grande è quella che farà finire il contesto
a metà, cioè esattamente il problema che questo metodo esiste per risolvere.

**Scrivi le dipendenze, non lasciarle intuire.** Ogni riga della tabella ha una colonna *Dipende da*:
ci vanno i numeri delle fasi che devono essere finite prima, e `—` quando non ce n'è nessuna. Non è
burocrazia: `/cantiere:veloce` la legge per capire quali fasi può mandare **insieme**
(`${CLAUDE_PLUGIN_ROOT}/PARALLELO.md`), e una dipendenza vera lasciata implicita è l'unico modo in
cui quel meccanismo può fare danno. Nel dubbio, la dipendenza si dichiara.

Le fasi indipendenti guadagnano se restano tali: quando puoi scegliere dove passa il taglio, taglia
per **file**, non per strato — due fasi che toccano cartelle diverse vanno in parallelo, due fasi che
si passano lo stesso file no, per quanto piccole siano.

Se il cantiere è lungo, mettici in fondo una **fase di refattorizzazione** (`/cantiere:refattorizza`)
prima della chiusura: `/cantiere:notturno` la inserisce da solo ogni cinque fasi, ma se il cantiere
lo esegui a mano è meglio che sia scritta.

### 5. Scrivi i file, con i nomi giusti

Apri `${CLAUDE_SKILL_DIR}/MODELLO.md` e segui quel formato. I nomi non si negoziano (veto,
regola 5):

| File | Cosa contiene | Tetto |
|---|---|---|
| `PIANO.md` | stato, decisioni, tabella delle fasi, forzature, scoperte | **4 KB** |
| `FASE_<NN>_<slug>.md` | una fase: obiettivo, file, cosa fare, verifica, trappole | 4 KB l'uno |

`<NN>` è il numero a due cifre (`01`, `02`, …), `<slug>` è il titolo in minuscolo con i trattini:
`FASE_03_integrazione-github.md`. In radice del progetto, accanto a `PIANO.md`.

Il tetto sul `PIANO.md` non è pignoleria: quel file viene letto **a ogni** sessione di esecuzione.
Ciò che non ci entra non si accorcia — si sposta nel file della fase, o in un
`PIANO_DOSSIER.md` (senza tetto, non lo apre nessuno se non serve) che la fase cita per paragrafo.

Scrivi i file **per sezioni**, non riscrivendoli da capo a ogni ripensamento.

### 6. Committa il piano, e fermati

Un commit solo, con i file del cantiere e nient'altro:

```bash
git add PIANO.md FASE_*.md
git commit -m "cantiere: piano aperto — <nome>"
git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

Poi **fermati** e di' all'utente, con queste parole:

> Il piano è scritto e committato: **<N> fasi**.
>
> **Apri una sessione nuova** e lancia `/cantiere:fase 1` (oppure `/cantiere:veloce` per andare
> spedito, o `/cantiere:notturno` per farle una all'ora da solo).
>
> Questa sessione ha in pancia tutta l'esplorazione: eseguire da qui vanificherebbe il metodo.

## Le tre cose che rovinano un piano

1. **«Nella zona di…»** invece di `file.ext:120`. Chi esegue riesplora, e il metodo non serve più.
2. **La cronaca mescolata ai passi.** Il piano dice cosa fare; com'è andata si scrive dopo, altrove.
3. **Una fase senza verifica.** Diventa «fatta» perché nessuno ha guardato.

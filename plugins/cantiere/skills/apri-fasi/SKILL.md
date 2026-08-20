---
name: apri-fasi
description: Apri il cantiere e crea un documento FASI.md con tutte le fasi strutturate in italiano (0, 1, 2, 3). Ogni fase avrà i dettagli di cosa fare e come verificare. Al termine di ogni fase, push e PR sono automatici.
argument-hint: [nome del progetto/cantiere]
disable-model-invocation: true
---

# /apri-fasi — Apri il cantiere con le fasi

Stai aprendo il cantiere: **$ARGUMENTS**

## 1. Crea il documento FASI.md

Crea un file `FASI.md` nella radice del progetto con la struttura sottostante. Questo file:
- Descrive tutte le 4 fasi in italiano
- Serve come guida per eseguire ogni fase
- Viene committato subito dopo la creazione

## 2. Struttura del documento FASI.md

Il documento deve contenere:
- **Fase 0**: Preparazione e setup
- **Fase 1**: Implementazione principale
- **Fase 2**: Testing e verifiche
- **Fase 3**: Finalizzazione e rilascio

Ogni fase deve avere:
- Descrizione chiara in italiano
- Obiettivi specifici
- File interessati (con ancore `file:riga`)
- Verifiche eseguibili
- Istruzioni per il push e la PR

## 3. Committa il documento

Un commit solo con messaggio: `cantiere: apri fasi per <nome>`

Esempio:
```
git add FASI.md
git commit -m "cantiere: apri fasi per <nome>"
git push -u origin <branch>
```

## 4. Istruzioni per l'esecuzione

Dopo aver committato FASI.md, la prossima azione è:

> Il cantiere è aperto. Lancia `/cantiere:fase 0` per iniziare con la Fase 0.
>
> **Importante**: Ogni fase deve essere completata in una sessione sola. Al termine di ogni fase:
> 1. Esegui la verifica
> 2. Committa il lavoro
> 3. Fai il push (automatico)
> 4. Crea la PR (automatica)
> 5. Apri una nuova sessione per la fase successiva

---

## Template FASI.md

Usa questo template per il file FASI.md:

```markdown
# Cantiere: <Nome del progetto>

**Fase corrente: 0 di 3** · **Ultimo commit: —** · **Stato: Aperto**

## Descrizione generale

<Descrizione breve del cantiere e dei suoi obiettivi>

## Fasi

### Fase 0: Preparazione e Setup
**Obiettivo**: Preparare l'ambiente e le strutture necessarie

**File interessati**:
- `file1.ts:10`
- `file2.md:25`

**Cosa fare** (in italiano):
1. <primo passo>
2. <secondo passo>
3. <verificare>

**Verifica**:
```
<comando da eseguire>
<cosa deve stampare/verificare>
```

**Trappole**: <avvertimenti se necessari>

---

### Fase 1: Implementazione Principale
**Obiettivo**: Implementare la funzionalità principale

**File interessati**:
- `file3.ts:50`
- `file4.md:100`

**Cosa fare** (in italiano):
1. <primo passo>
2. <secondo passo>
3. <verificare>

**Verifica**:
```
<comando da eseguire>
<cosa deve stampare/verificare>
```

**Trappole**: <avvertimenti se necessari>

---

### Fase 2: Testing e Verifiche
**Obiettivo**: Testare e verificare la funzionalità

**File interessati**:
- `test1.spec.ts:1`
- `coverage/`

**Cosa fare** (in italiano):
1. <primo passo>
2. <secondo passo>
3. <verificare>

**Verifica**:
```
<comando da eseguire>
<cosa deve stampare/verificare>
```

**Trappole**: <avvertimenti se necessari>

---

### Fase 3: Finalizzazione e Rilascio
**Obiettivo**: Finalizzare e preparare al rilascio

**File interessati**:
- `package.json:1`
- `CHANGELOG.md:1`

**Cosa fare** (in italiano):
1. <primo passo>
2. <secondo passo>
3. <verificare>

**Verifica**:
```
<comando da eseguire>
<cosa deve stampare/verificare>
```

**Trappole**: <avvertimenti se necessari>

---

## Note per l'esecuzione

- **Una fase per sessione**: Completa una fase in una sessione sola, niente di più, niente di meno.
- **Verifica sempre**: Non passare oltre finché la verifica non passa.
- **Nuova sessione per ogni fase**: Apri una sessione fresca per ogni fase successiva.
- **Push e PR automatici**: Alla fine di ogni fase, il push e la PR sono automatici.

## Scoperte durante l'esecuzione

<Vuoto all'inizio. Si compila quando si trovano cose inaspettate.>
```

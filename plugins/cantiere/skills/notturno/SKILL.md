---
name: notturno
description: Cantiere notturno. Un giro all'ora - se una fase è già in corso non fa nulla, altrimenti esegue la fase successiva, la verifica e la committa. Ogni cinque fasi consecutive inserisce una refattorizzazione con le linee guida di Robert Martin, e alla fine passa alla fase conclusiva. Lavora da solo, senza fare domande.
disable-model-invocation: true
model: opus
effort: max
disallowed-tools: AskUserQuestion
---

# /cantiere:notturno — un giro all'ora, da solo

ultrathink

Questa skill è **un giro**, non un turno intero. Ogni invocazione fa una cosa sola, la committa e
riarma il giro dopo. Il ciclo è quello:

```
ogni ora → c'è una fase in corso? → sì: non fare nulla
                                  → no: fai la prossima cosa, committa, pusha
```

Di notte non c'è nessuno da svegliare: **non fai domande** (`AskUserQuestion` è disattivato apposta)
e non tiri a indovinare. Quando serve una decisione, ti fermi e la lasci scritta.

## Come si arma il ciclo

Il modo previsto è il ciclo di Claude Code, che rilancia questa skill da solo:

```
/loop 1h /cantiere:notturno
```

Se la sessione ha strumenti di pianificazione (attività programmate, `send_later`, `CronCreate`),
usali per fissare il giro dopo a **un'ora** da adesso. Se non c'è né l'uno né gli altri, fai il giro
di adesso e chiudi dicendo all'utente di lanciare `/loop 1h /cantiere:notturno`: un giro solo è
comunque un giro fatto.

## Il registro

Lo stato del ciclo sta in `.cantiere/NOTTURNO.md` (fuori da git, come tutto ciò che sta in
`.cantiere/`). Se non c'è, crealo:

```markdown
# Cantiere notturno

**Avviato:** <data e ora> · **Fasi consecutive dall'ultima refattorizzazione:** 0 · **Stato:** attivo

## Giri

| Ora | Cosa ho fatto | Esito |
|---|---|---|
```

Una riga per giro, sempre, anche — soprattutto — quando il giro non fa niente.

## Il giro, passo per passo

### 1. C'è una fase in corso?

Fai i tre controlli di `${CLAUDE_PLUGIN_ROOT}/LOCK.md`: segnale `.cantiere/IN-CORSO`, copia di lavoro sporca, ramo
remoto avanti con una fase `🔨 in corso`.

**Se una fase è in corso: non fai nulla.** Scrivi la riga nel registro («fase 03 in corso da
22:14, salto»), riarma il giro dopo, chiudi il turno. Non è un errore: è il caso che questo
controllo esiste per gestire. L'unica eccezione è il segnale scaduto (più di tre ore *e* copia di
lavoro pulita): quello lo rilasci, come dice `LOCK.md`, e prosegui.

### 2. Cosa tocca fare, in questo giro

Leggi la riga di stato e la tabella delle fasi di `PIANO.md`, e il contatore nel registro. Poi,
**in quest'ordine**:

| Se… | Il giro è |
|---|---|
| il contatore è **≥ 5** | **refattorizzazione** — segui `${CLAUDE_PLUGIN_ROOT}/skills/refattorizza/SKILL.md`, poi rimetti il contatore a 0 |
| c'è una fase non `✅ COMPLETA` | **la prossima fase** — la prima non completa della tabella; contatore +1 |
| tutte le fasi sono complete e il contatore è **> 0** | **refattorizzazione** — l'ultima passata prima di chiudere; contatore a 0 |
| tutte le fasi sono complete e il contatore è **0** | **la fase conclusiva** — segui `${CLAUDE_PLUGIN_ROOT}/skills/chiudi/SKILL.md` |

Cinque fasi consecutive senza una passata di pulizia sono cinque fasi che nessuno ha riletto: la
refattorizzazione non è un premio a fine cantiere, è manutenzione, e va fatta finché il codice è
ancora fresco.

### 3. Fai la cosa, e falla per intero

Prendi il segnale (`${CLAUDE_PLUGIN_ROOT}/LOCK.md`), poi esegui **una cosa sola**: la fase, o la passata di
refattorizzazione, o la chiusura. Le regole sono quelle di sempre — solo i file che la fase nomina,
`lettore` per cercare, niente allargamenti (`${CLAUDE_PLUGIN_ROOT}/VETO.md`).

Poi **controlla**, come farebbe `/cantiere:veloce`: la verifica della fase, i controlli veloci del
progetto, e una riletta del diff da avversario. Sul rosso non si committa.

### 4. Committa, pusha, rilascia

Segna la fase `✅ COMPLETA` in `PIANO.md` e `**Stato: COMPLETA**` nel suo file, poi:

```bash
git add -A
git commit -m "cantiere fase <NN>: <titolo>"      # oppure: cantiere: refattorizzazione (Clean Code) — <zona>
git push -u origin $(git rev-parse --abbrev-ref HEAD)
rm -f .cantiere/IN-CORSO
```

Push con quattro tentativi e attese di 2s, 4s, 8s, 16s. **Un giro che non finisce con un commit
pushato è un giro perso**: il container di stanotte domattina non c'è più.

Aggiorna il registro (riga del giro, contatore) e riarma il giro dopo.

### 5. Quando ti fermi

Il ciclo si ferma da solo in tre casi, e in tutti e tre **lo scrivi nel registro e lo dici
nell'ultimo messaggio**:

1. **Il cantiere è chiuso.** `/cantiere:chiudi` è andato a buon fine: niente più giri.
2. **Serve una decisione.** Una fase non regge, la verifica è rossa per un motivo che non sai
   sistemare senza scegliere, il veto scatta su qualcosa che solo l'utente può forzare. Allora:
   scrivi cosa hai trovato e cosa serve decidere in **Scoperte durante l'esecuzione** di `PIANO.md`,
   committa e pusha quella sola modifica, metti `**Stato:** fermo — serve una decisione` in cima al
   registro, e chiudi il ciclo. Non tirare a indovinare di notte.
3. **La verifica fallisce due giri di fila sulla stessa fase.** Non insistere una terza volta:
   scrivi cosa hai provato, e fermati come al punto 2.

Se invece il contesto della sessione si sta riempiendo, chiudi il giro dicendo:

> Giro fatto e committato. Il contesto è quasi pieno: `/clear` e poi
> `/loop 1h /cantiere:notturno` per riprendere il ciclo da una sessione pulita.

## Il rapporto del mattino

Quando il ciclo finisce, l'ultimo messaggio dice, in poche righe: quante fasi sono state fatte,
quali sono state saltate e perché, se c'è stata una refattorizzazione, com'è finito il cantiere, e
cosa resta da decidere. È l'unica cosa che l'utente leggerà a colazione: mettici dentro quello che
serve per decidere, non la cronaca dei giri.

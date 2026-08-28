---
name: notturno
description: Cantiere notturno. Un giro all'ora - se una fase è già in corso non fa nulla, altrimenti manda l'agente esecutore sulla fase successiva e ne controlla l'esito. Ogni cinque fasi consecutive inserisce una refattorizzazione con le linee guida di Robert Martin, e alla fine passa alla fase conclusiva. Lavora da solo, senza fare domande.
disable-model-invocation: true
model: sonnet
effort: medium
disallowed-tools: AskUserQuestion
---

# /cantiere:notturno — un giro all'ora, da solo

Questa skill è **un giro**, non un turno intero. Ogni invocazione fa una cosa sola, la controlla e
riarma il giro dopo. Il ciclo è quello:

```
ogni ora → c'è una fase in corso? → sì: non fare nulla
                                  → no: manda l'esecutore, controlla, riarma
```

Di notte non c'è nessuno da svegliare: **non fai domande** (`AskUserQuestion` è disattivato apposta)
e non tiri a indovinare. Quando serve una decisione, ti fermi e la lasci scritta.

E **il lavoro non lo fai tu**: ogni cosa la esegue l'agente **`esecutore`**, in un contesto suo.
Non è solo risparmio: è ciò che permette al ciclo di andare avanti per ore nella stessa sessione
senza che nessuno debba passare a premere `/clear`. Se ti metti a leggere i file delle fasi, la
notte finisce al quinto giro.

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

Fai i tre controlli di `${CLAUDE_PLUGIN_ROOT}/LOCK.md`: segnale `.cantiere/IN-CORSO`, copia di lavoro
sporca, ramo remoto avanti con una fase `🔨 in corso`.

**Se una fase è in corso: non fai nulla.** Scrivi la riga nel registro («fase 03 in corso da
22:14, salto»), riarma il giro dopo, chiudi il turno. Non è un errore: è il caso che questo
controllo esiste per gestire. L'unica eccezione è il segnale scaduto (più di tre ore *e* copia di
lavoro pulita): quello lo rilasci, come dice `LOCK.md`, e prosegui.

### 2. Cosa tocca fare, in questo giro

Leggi la riga di stato e la tabella delle fasi di `PIANO.md`, e il contatore nel registro. Poi,
**in quest'ordine**:

| Se… | Il giro è |
|---|---|
| il contatore è **≥ 5** | **refattorizzazione** — le linee guida stanno in `${CLAUDE_PLUGIN_ROOT}/skills/refattorizza/SKILL.md`, poi rimetti il contatore a 0 |
| c'è una fase non `✅ COMPLETA` | **la prossima fase** — la prima non completa della tabella; contatore +1 |
| tutte le fasi sono complete e il contatore è **> 0** | **refattorizzazione** — l'ultima passata prima di chiudere; contatore a 0 |
| tutte le fasi sono complete e il contatore è **0** | **la fase conclusiva** — segui `${CLAUDE_PLUGIN_ROOT}/skills/chiudi/SKILL.md` |

Cinque fasi consecutive senza una passata di pulizia sono cinque fasi che nessuno ha riletto: la
refattorizzazione non è un premio a fine cantiere, è manutenzione, e va fatta finché il codice è
ancora fresco.

### 3. Manda l'esecutore

Prima, **una riga che dice a che punto sei** — nel registro e nel messaggio del giro:

```
**Fase 3 di 7** — Integrazione GitHub
```

Per una passata di pulizia, `**Refattorizzazione** — dopo la fase 5 di 7`. Vale a ogni giro: la
mattina, il rapporto si legge molto meglio se ogni giro dice da solo quanto mancava.

**Una cosa sola per giro**, e la fa lui: niente gruppi paralleli, quelli sono di `/cantiere:veloce`.
Qui il ritmo è un'ora per giro, e un giro fa una cosa. L'incarico è corto: non ha visto niente di questa sessione,
ma legge i file da solo.

Per una fase:

> Esegui la fase **<NN>** del cantiere in questo repository. Leggi `PIANO.md` (riga di stato,
> Decisioni) e `FASE_<NN>_<slug>.md`, fai il lavoro, lancia la verifica, segna la fase completa,
> committa e pusha. Rispondi nella forma prevista. Non fare domande: se serve una decisione,
> scrivila in **Scoperte durante l'esecuzione** e fermati.

Per la refattorizzazione:

> Fai una passata di refattorizzazione sul codice toccato dal cantiere, seguendo
> `${CLAUDE_PLUGIN_ROOT}/skills/refattorizza/SKILL.md` alla lettera. I test devono essere verdi
> prima e dopo, il comportamento non cambia, e il commit è separato. Rispondi nella forma prevista.

La **fase conclusiva** invece falla tu, seguendo `${CLAUDE_PLUGIN_ROOT}/skills/chiudi/SKILL.md`: lì
si cancellano file e si scrive nel diario del progetto, ed è l'unica cosa della notte che non si
delega. Se qualcosa è rosso, non si cancella niente.

### 4. Controlla che sia vero

Il rapporto dell'esecutore non è una prova. Quattro comandi, che costano poco:

```bash
git status --porcelain                      # vuoto: niente lasciato a metà
git log -1 --oneline                        # il commit del giro
git log @{u} -1 --oneline                   # lo stesso sha: è pushato davvero
grep -n 'FASE_<NN>' PIANO.md                # la riga dice ✅ COMPLETA
```

Poi **rilancia la verifica della fase** e guarda l'esito. **Un giro che non finisce con un commit
pushato è un giro perso**: il container di stanotte domattina non c'è più.

Aggiorna il registro (riga del giro, contatore) e riarma il giro dopo.

### 5. Quando ti fermi

Il ciclo si ferma da solo in tre casi, e in tutti e tre **lo scrivi nel registro e lo dici
nell'ultimo messaggio**:

1. **Il cantiere è chiuso.** La fase conclusiva è andata a buon fine: niente più giri.
2. **Serve una decisione.** L'esecutore ha risposto `FERMATA`, una fase non regge, il veto scatta su
   qualcosa che solo l'utente può forzare. Allora: scrivi cosa hai trovato e cosa serve decidere in
   **Scoperte durante l'esecuzione** di `PIANO.md`, committa e pusha quella sola modifica, metti
   `**Stato:** fermo — serve una decisione` in cima al registro, e chiudi il ciclo. Non tirare a
   indovinare di notte.
3. **La verifica fallisce due giri di fila sulla stessa fase.** Non mandare un terzo esecutore:
   scrivi cosa è stato provato, e fermati come al punto 2.

Il contesto di questa sessione cresce di poche righe a giro, perché il lavoro sta tutto dentro
l'esecutore: se malgrado questo si sta riempiendo, non consegnare comandi da premere — scrivi lo
stato nel registro, committa, e chiudi il ciclo dicendo che va rilanciato.

## Il rapporto del mattino

Quando il ciclo finisce, l'ultimo messaggio dice, in poche righe: quante fasi sono state fatte,
quali sono state saltate e perché, se c'è stata una refattorizzazione, com'è finito il cantiere, e
cosa resta da decidere. È l'unica cosa che l'utente leggerà a colazione: mettici dentro quello che
serve per decidere, non la cronaca dei giri.

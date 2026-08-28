---
name: veloce
description: Cantiere veloce. Esegue la fase successiva, la ricontrolla, salva tutto, committa e pusha, poi ti consegna il /clear pronto per la fase dopo. Come /cantiere:fase, ma senza la PR a ogni giro e con un controllo in più prima del commit.
argument-hint: "[numero della fase]"
disable-model-invocation: true
model: opus
effort: max
---

# /cantiere:veloce — un passo, controllato, salvato, e via

Fase: **$ARGUMENTS** (se è vuoto, la prima non `✅ COMPLETA` della tabella di `PIANO.md`).

ultrathink

Il giro è sempre lo stesso: **una fase → il controllo → il salvataggio → `/clear`**. La PR non si
apre a ogni fase: qui conta la velocità, e la PR la fa `/cantiere:chiudi` alla fine (o
`/cantiere:fase`, se ti serve subito). Se una PR è già aperta, il push la aggiorna da solo.

## 1. Parti

Leggi solo la riga di stato e la tabella di `PIANO.md`, le **Decisioni**, e il file
`FASE_<NN>_<slug>.md` della fase. Niente fasi successive.

Segui `${CLAUDE_PLUGIN_ROOT}/LOCK.md`: se una fase è già in corso, fermati e chiedi. Altrimenti prendi il segnale e
porta la fase a `🔨 in corso`.

## 2. Fai il passo

Esegui **solo quella fase**, sui file che nomina. Per «dove sta / com'è fatto» chiama l'agente
`lettore`. Niente allargamenti: le cose che scopri e che non c'entrano vanno in **Scoperte durante
l'esecuzione**, non nel diff (veto, regola 8 — `${CLAUDE_PLUGIN_ROOT}/VETO.md`).

## 3. Controllalo (questo è il passo che gli altri saltano)

Prima di committare, tre controlli, in quest'ordine:

1. **La verifica della fase**: lanciala e guarda l'esito. Rossa = fase non fatta.
2. **I controlli veloci del progetto**: lint, format, typecheck, i test del pezzo toccato — quelli
   che un umano lancia prima di committare. Se il progetto ne ha, si lanciano.
3. **Rileggi il tuo diff** (`git diff`) da avversario: cosa ci hai lasciato dentro che non
   c'entra? Un file di prova, una stampa di debug, una riga commentata, un file che la fase non
   nominava? Toglilo adesso.

Se qualcosa è rosso, correggi e ricomincia da 1. **Non si committa sul rosso.**

## 4. Salva tutto

Segna la fase completa — `**Stato: COMPLETA**` più lo sha nel file della fase, `✅ **COMPLETA**` e
riga di stato aggiornata in `PIANO.md` — e committa tutto insieme:

```bash
git add -A
git commit -m "cantiere fase <NN>: <titolo della fase>"
git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

Push con quattro tentativi e attese di 2s, 4s, 8s, 16s se la rete fa i capricci. Poi rilascia il
segnale: `rm -f .cantiere/IN-CORSO`.

**Salvato vuol dire pushato**: un commit che sta solo nel container non è salvato.

## 5. Consegna il /clear

Chiudi con tre righe — cosa hai fatto, cosa ha stampato la verifica, qual è la prossima fase — e poi
esattamente questo, perché `/clear` lo può premere solo l'utente:

> **Fase <NN> fatta, controllata, committata e pushata.** ✅
>
> Ora, in quest'ordine:
>
> ```
> /clear
> /cantiere:veloce
> ```
>
> Il contesto riparte vuoto e la fase <N+1> comincia pulita.

Se era l'ultima fase, la seconda riga diventa `/cantiere:chiudi`.

**Non cominciare la fase successiva in questa sessione**, nemmeno se l'utente insiste che è piccola:
è il contesto sporco della fase precedente il motivo per cui esiste il `/clear`.

---
name: refattorizza
description: Passata di refattorizzazione sul codice toccato dal cantiere, con le linee guida di Robert C. Martin (Clean Code, SOLID, regola del boy scout). Non cambia il comportamento, e i test devono essere verdi prima e dopo. La fa fare da sola /cantiere:notturno ogni cinque fasi, mandandoci l'agente esecutore.
argument-hint: "[zona da rifattorizzare]"
disable-model-invocation: true
model: sonnet
effort: medium
---

# /cantiere:refattorizza — la passata di Robert Martin

Zona: **$ARGUMENTS** (se è vuoto, il codice toccato dalle fasi del cantiere: `git diff` rispetto al
ramo base, o gli ultimi commit `cantiere fase`).

Quando è il cantiere a chiedere la passata — `/cantiere:notturno` ogni cinque fasi — queste stesse
righe le esegue l'agente `esecutore` nel suo contesto. Lanciata a mano, invece, la passata la fai
qui: è corta e sta in una volta sola.

## La regola che viene prima di tutte

**Refattorizzare non cambia il comportamento.** Se il comportamento cambia, non è una
refattorizzazione: è una modifica, e va in una fase sua, con il suo piano e la sua verifica.

Quindi, in quest'ordine, sempre:

1. **I test sono verdi prima.** Se sono rossi, non si refattorizza: si ferma tutto e si dice perché.
   Se la zona non ha test e la refattorizzazione non è banale, scrivi prima il test che fissa il
   comportamento di adesso — è quello che rende sicuro tutto il resto.
2. Un passo per volta, piccolo, con i test lanciati dopo ognuno.
3. **I test sono verdi dopo**, e sono *gli stessi* test: se hai dovuto cambiarne uno per farlo
   passare, hai cambiato il comportamento. Torna indietro.

## Cosa guardi, nell'ordine in cui conviene guardarlo

**Nomi.** Un nome deve dire l'intenzione: cosa fa, perché esiste, come si usa. `d` non è un nome,
`getData` nemmeno. Niente sigle interne, niente `Manager`/`Helper`/`Utils` come discarica. Se per
capire un nome devi leggere il corpo, il nome è sbagliato.

**Funzioni.** Piccole, e poi più piccole. **Una funzione fa una cosa sola**, a un solo livello di
astrazione (leggerla deve somigliare a leggere una frase, non a scendere e risalire una scala).
Pochi parametri: zero è meglio di uno, tre sono già tanti. **Niente parametri-bandiera**: un
booleano che sceglie fra due comportamenti sono due funzioni. **Comando o domanda, mai entrambi**:
o cambia lo stato, o risponde a una domanda.

**Duplicazione.** È il peccato capitale: la stessa regola scritta in due posti diverge il giorno che
qualcuno ne cambia una sola. Ma tre righe che si somigliano non sono duplicazione — la duplicazione
è di *concetto*, non di caratteri.

**Effetti collaterali.** Una funzione che promette una cosa e ne fa anche un'altra di nascosto è una
bugia, e le bugie nel codice costano care.

**Commenti.** Un commento che spiega *cosa* fa il codice è un fallimento del codice: rendilo
leggibile e cancella il commento. Restano i commenti che spiegano il *perché* — la scelta strana, il
vincolo esterno, il bug che quella riga evita. Il codice commentato via si cancella: c'è git.

**Gestione degli errori.** Errori separati dalla logica; niente `null` restituiti che poi ognuno
controlla a modo suo; niente `catch` vuoti che ingoiano il problema.

**SOLID**, dove ha senso davvero, non per compitino:

| | In una riga |
|---|---|
| **S** — responsabilità singola | Una classe, un motivo per cambiare. Se due motivi diversi la toccano, sono due classi. |
| **O** — aperto/chiuso | Estendere senza dover riaprire ciò che già funziona. |
| **L** — sostituzione di Liskov | Un sottotipo si usa al posto del suo tipo senza sorprese. |
| **I** — segregazione delle interfacce | Meglio due interfacce piccole di una grande che nessuno usa tutta. |
| **D** — inversione delle dipendenze | Si dipende da astrazioni, non da dettagli. |

**Test.** Valgono le stesse regole del codice di produzione: nomi che dicono cosa verificano, un
concetto per test, niente copia-incolla. Un test illeggibile smette di essere manutenuto, e un test
non manutenuto smette di proteggere.

**La regola del boy scout.** Lascia il campo più pulito di come l'hai trovato — ma *il campo*: la
zona del cantiere, non tutto il repository. Una refattorizzazione che tocca quaranta file non si
rivede, quindi non si fida.

## Quello che non fai

- **Non allarghi il perimetro.** Solo il codice toccato dal cantiere (o la zona che ti è stata
  indicata). Il resto lo vedi, lo annoti in **Scoperte durante l'esecuzione**, e lo lasci stare.
- **Non riscrivi da capo.** Riscrivere non è refattorizzare: è una fase nuova, con un piano suo.
- **Non cambi le interfacce pubbliche** senza dirlo: rinominare una cosa che qualcun altro chiama è
  una modifica di comportamento per chi la chiama.
- **Non mescoli** refattorizzazione e funzionalità nello stesso commit. Mai. Un diff misto è un diff
  che nessuno rivede davvero.

## Come chiudi

Test verdi, poi un commit **separato**, che si riconosce a colpo d'occhio:

```bash
git add -A
git commit -m "cantiere: refattorizzazione (Clean Code) — <zona>"
git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

Nel corpo del commit, l'elenco di cosa hai cambiato e perché: «estratta `validaOrario` da
`Prenotazione.salva` (faceva due cose)», «rinominato `flag` in `inviaNotifica`», una riga l'una.

In `PIANO.md`, una riga in **Scoperte durante l'esecuzione**: cosa hai pulito e cosa hai lasciato
lì apposta.

Poi ti fermi: la refattorizzazione è una passata, non un turno di lavoro infinito.

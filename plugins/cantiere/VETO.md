# Il diritto di veto del cantiere

Il cantiere **non è un esecutore di ordini**: è un metodo, e un metodo che si piega a ogni
richiesta non serve a niente. Per questo il plugin ha **diritto di veto su come si fa il piano**.

Il veto vale in `/cantiere:piano`, e vale anche dopo: se durante una fase salta fuori che il piano
viola una regola, la fase si ferma e il veto si applica lì.

## Le nove regole non negoziabili

1. **Nessuna fase senza verifica eseguibile.** Un comando, un test, una schermata da guardare. «Controllare che funzioni» non è una verifica: se non sai dire come si vede che è fatto, non è una fase.
2. **Una fase = un contesto = un commit.** Il contesto è quello di chi la esegue: l'agente `esecutore`, che parte pulito e non vede nient'altro. Se una fase non ci sta dentro, o produce un commit che non ha senso da solo, va spezzata prima di essere scritta. Due fasi insieme, o due esecutori in parallelo, sono la stessa violazione.
3. **Ancore, non descrizioni.** `percorso/file.ext:120`, mai «nella zona del pulsante». Un riferimento vago costringe chi esegue a riesplorare, e riesplorare è esattamente il costo che questo metodo esiste per evitare.
4. **Nessuna fase può dipendere da scoperte di una fase successiva.** L'ordine delle fasi è un ordine vero, non una lista.
5. **I nomi sono quelli, sempre.** `PIANO.md` in radice, `FASE_<NN>_<slug>.md` accanto. Niente `FASI.md`, niente `piano-v2-definitivo.md`, niente nomi inventati per l'occasione.
6. **Chi pianifica non esegue.** La sessione che scrive il piano ha in pancia tutta l'esplorazione: eseguire da lì vanifica il metodo. Nemmeno la prima fase, nemmeno se è banale — e nemmeno delegandola all'esecutore: il contesto che coordina resterebbe comunque quello gonfio della pianificazione.
7. **Una fase finita si committa.** Sempre, prima di fermarsi. Lavoro non committato è lavoro perso, e in una sessione cloud lo è davvero.
8. **Niente allargamenti.** Pulizie, rinomine e miglioramenti non chiesti non entrano in una fase: si scrivono in **Scoperte durante l'esecuzione** e si decidono con calma.
9. **Un cantiere finito si smonta.** All'ultima fase i file del cantiere si cancellano e il perché si condensa nel diario del progetto. Un `PIANO.md` con «Stato: fatto» lasciato in giro è peso che ogni sessione futura si porta dietro senza usarlo mai.

## Come si esercita il veto

Quando la richiesta viola una regola, **non la esegui e non la aggiusti in silenzio**. Dici, in
questa forma:

```
VETO — regola <n>: <la regola, in mezza riga>
Perché qui: <una riga sul caso concreto>
Proposta: <la versione che passa il veto>
```

Poi ti fermi e aspetti. Non scrivere il piano nella forma vietata «intanto che ne parliamo».

## Come si forza il veto

Il veto è forte, non è una prigione: **decide l'utente**. Se l'utente risponde esplicitamente di
procedere lo stesso — «forza», «procedi così», «va bene lo stesso» — allora si procede, e la
forzatura **si scrive nel piano**, nella tabella *Veto e forzature* di `PIANO.md`:

| # | Regola forzata | Cosa è stato chiesto | Motivo dato |

Serve a due cose: chi eseguirà la fase sa che quella stranezza è voluta, e fra tre settimane si sa
perché. Un veto forzato e non scritto è un veto che non è mai esistito.

Non basta il silenzio, e non basta l'insistenza sullo stesso testo: serve una risposta che dice di
procedere. Se l'utente non risponde alla contestazione, il veto tiene.

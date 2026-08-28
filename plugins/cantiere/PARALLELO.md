# Il gruppo parallelo — quando due fasi possono andare insieme

Il cantiere è fatto di fasi in fila, e la fila è la scelta giusta quasi sempre: l'ordine delle fasi
è un ordine vero (veto, regola 4), e una fase che aspetta il suo turno non ha mai rotto niente.

Ma un piano non è una catena tutta intera: certi pezzi non si toccano fra loro. Due fasi che
scrivono file diversi, che non si citano, e che si verificano da sole possono andare **insieme** —
due esecutori, due contesti puliti, un giro solo. È l'unico posto del cantiere dove succede, e ci
succede solo alle condizioni scritte qui.

Chi lo usa: **`/cantiere:veloce`**, che è la skill fatta per macinare. `/cantiere:fase` fa una fase
per definizione, e `/cantiere:notturno` fa una cosa per giro: nessuna delle due parallelizza.

## Le quattro condizioni

Perché due fasi stiano nello stesso gruppo devono valere **tutte e quattro**. Ne manca una, e vanno
in fila come sempre.

1. **File disgiunti.** Le sezioni `## File` delle due fasi non hanno un percorso in comune — nemmeno
   uno che la prima *crea* e la seconda *modifica*. Stesso file, stesso giro: mai, e non c'è dubbio
   da sciogliere. Vale anche per i file condivisi che il piano non nomina ma che il lavoro tocca di
   sicuro: un file di rotte, un `index` che raccoglie gli export, un file di migrazioni numerate.
2. **Nessuna dipendenza.** Nessuna delle due compare nella colonna *Dipende da* dell'altra, e nessuna
   nomina l'altra per numero nel suo `Cosa fare` o nelle sue `Trappole`. Una fase che comincia con
   «ora che la fase 2 ha creato il servizio» non è parallelizzabile, e non conta che sembri piccola.
3. **Verifiche che convivono.** I due comandi di verifica devono poter girare **nello stesso momento
   nella stessa copia di lavoro**: niente stessa porta, stesso database, stessa cartella di build,
   stesso file di lock del gestore di pacchetti. Se per verificarle serve che una finisca prima
   dell'altra, sono in fila.
4. **Sono fasi normali.** Una passata di `/cantiere:refattorizza` non entra mai in un gruppo: tocca
   il codice di tutte le fasi per definizione. E non entra una fase il cui lavoro è il piano stesso.

**Nel dubbio, in fila.** Il parallelo è un'ottimizzazione: la fila è sempre corretta, il gruppo
sbagliato no. Se per decidere devi aprire i file del codice, hai già la risposta — non lo era.

## Come si forma il gruppo

Il gruppo si costruisce **dalla fase corrente in avanti**, prendendo le successive finché reggono le
quattro condizioni contro *tutte* quelle già dentro; alla prima che non regge, il gruppo si chiude
lì. Così non restano buchi nella tabella e l'ordine del piano resta leggibile.

**Al massimo tre fasi per gruppo.** Non è un limite tecnico: è che il commit di un gruppo deve
restare rivedibile, e quattro fasi insieme non lo sono più.

Un gruppo di una fase sola è il caso normale, non un fallimento dell'analisi.

## Come gira un gruppo

Le regole del cantiere non cambiano: cambia **chi** fa cosa. In un gruppo il caposquadra tiene la
contabilità che in fila teneva l'esecutore, ed è quello che evita che due esecutori si pestino i
piedi su `PIANO.md` e sul commit.

1. **Il segnale lo prende il caposquadra**, una volta per il gruppo (`${CLAUDE_PLUGIN_ROOT}/LOCK.md`),
   scrivendoci dentro tutte le fasi: `fasi: 03,04`. Porta a `🔨 in corso` le righe del gruppo in
   `PIANO.md` e nei file delle fasi.
2. **Gli esecutori partono insieme**, in **modalità parallela**: le chiamate vanno nello stesso
   messaggio, altrimenti non sono parallele — partono una dopo l'altra e hai solo complicato la fila.
3. **Ogni esecutore fa il lavoro e la verifica, e si ferma lì**: segna `**Stato: COMPLETA**` e
   compila **Fatto** solo nel *suo* `FASE_<NN>_<slug>.md`. Non tocca `PIANO.md`, non prende il
   segnale, **non committa e non pusha**. Lo scrive l'incarico, non se lo ricorda da solo.
4. **Il caposquadra controlla il gruppo**: che i file rimasti sporchi siano **solo** quelli dichiarati
   dalle fasi del gruppo (più i loro file di fase), e poi rilancia le verifiche **una per volta**.
   Un file sporco che nessuna fase nominava è una collisione o un allargamento: si ferma lì.
5. **Un commit solo per il gruppo**, fatto dal caposquadra, con dentro il lavoro di tutte le fasi e i
   due aggiornamenti di stato:

   ```bash
   git add -A
   git commit -m "cantiere fasi 03+04: <titolo>, <titolo>"
   git push -u origin $(git rev-parse --abbrev-ref HEAD)
   rm -f .cantiere/IN-CORSO
   ```

   Lo sha nei file delle fasi non lo scrive nessuno: il commit del gruppo è uno solo e sta nel log —
   nel file della fase restano lo stato e il **Fatto**.

## Se una fase del gruppo cade

Le fasi del gruppo hanno file disgiunti, ed è proprio questo che permette di salvare il salvabile
senza districare niente:

- **Una verde, una rossa** → committa **solo** i file dichiarati dalla fase verde e il suo file di
  fase (`git add` con i percorsi, non `-A`), lascia sporco il lavoro della rossa, e fermati come per
  una fase rossa qualunque: cosa è rosso, scritto in **Scoperte durante l'esecuzione**.
- **Tutte rosse** → non committi niente del lavoro, e ti fermi.
- **Una `FERMATA`** → il gruppo si ferma con lei: la domanda va portata all'utente prima di andare
  avanti, anche se l'altra fase è finita bene.

E in ogni caso: **non si rimanda lo stesso incarico una terza volta**, e non si riprova in fila una
fase caduta in parallelo sperando che vada meglio. Se è caduta, c'è un motivo da scrivere.

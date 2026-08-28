# «C'è una fase in corso?» — il segnale del cantiere

Due sessioni che lavorano sulla stessa fase si pestano i piedi e producono un commit illeggibile.
Per questo, prima di aprire una fase, si guarda se ce n'è una in corso. Il segnale è fatto di tre
controlli, dal più economico al più lento.

## Prendere il segnale (prima di toccare qualunque file)

```bash
mkdir -p .cantiere
printf 'fase: %s\navviata: %s\nramo: %s\n' "<NN>" "$(date -Iseconds)" "$(git rev-parse --abbrev-ref HEAD)" > .cantiere/IN-CORSO
grep -qxF '.cantiere/' .gitignore 2>/dev/null || echo '.cantiere/' >> .gitignore
```

Nello stesso momento, in `PIANO.md` e nel file della fase, lo stato passa a **in corso**
(`🔨 in corso` nella tabella, `**Stato: IN CORSO**` nel file). Questo non si committa da solo:
viaggia nel commit di fine fase.

## Leggere il segnale (i tre controlli)

1. **`.cantiere/IN-CORSO` esiste** → una fase è aperta in questa macchina. Leggi quale e da quando.
2. **La copia di lavoro non è pulita** (`git status --porcelain` stampa qualcosa) → una fase è a
   metà: il lavoro c'è ma non è committato. Vale come «in corso» anche senza il file.
3. **`PIANO.md` dice `🔨 in corso`** su una fase, e `git fetch` mostra il ramo remoto avanti
   rispetto a te → sta lavorando **un'altra sessione**, altrove.

## Cosa fare se il segnale c'è

- `/cantiere:fase` e `/cantiere:veloce`: **fermati e chiedi.** Non riaprire una fase aperta: o la
  finisce chi l'ha aperta, o l'utente dice esplicitamente di riprenderla.
- `/cantiere:notturno`: **non fare nulla.** Scrivi una riga nel registro, riarma il giro dopo,
  chiudi il turno. È il caso previsto, non un errore.

## Segnale vecchio

Se `.cantiere/IN-CORSO` è più vecchio di **tre ore** *e* la copia di lavoro è pulita, la sessione
che l'aveva preso non c'è più (container riciclato, sessione chiusa a metà). Allora:

```bash
rm -f .cantiere/IN-CORSO
```

e scrivilo nel registro: «segnale scaduto, fase <NN> mai chiusa, rilascio». Con la copia di lavoro
**sporca** invece non si rilascia mai: lì dentro c'è lavoro di qualcuno.

## Rilasciare il segnale

Solo dopo che il commit di fine fase è andato a buon fine:

```bash
rm -f .cantiere/IN-CORSO
```

Il segnale si rilascia **dopo** il commit, mai prima: se qualcosa va storto nel mezzo, meglio un
segnale di troppo che due sessioni sulla stessa fase.

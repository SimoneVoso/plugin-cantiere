# Fase 1: Automazione Push/PR

**Cantiere**: Fasi Markdown  
**Fase**: 1 di 5  
**Status**: 🔄 In corso  
**Data inizio**: 2026-08-20

---

## 📌 Obiettivo

Implementare l'automazione completa di **push automatico** e **creazione/aggiornamento PR automatica** nella skill `/cantiere:fase`.

## 📋 Checklist

- [ ] Implementare push automatico con retry esponenziale
- [ ] Implementare creazione PR tramite GitHub MCP tools
- [ ] Implementare aggiornamento PR per fasi successive
- [ ] Testare push fallito e recovery
- [ ] Testare creazione e aggiornamento PR
- [ ] Documentare il flusso nel SKILL.md

## 🔧 File da modificare

| File | Riga | Cosa fare |
|------|------|-----------|
| `plugins/cantiere/skills/fase/SKILL.md` | 80-150 | Aggiungere logica push con retry |
| `plugins/cantiere/skills/fase/SKILL.md` | 150-200 | Aggiungere logica creazione/update PR |
| `plugins/cantiere/skills/fase/SKILL.md` | 200-250 | Aggiungere messaggi di feedback |

## ✅ Verifica

Dopo aver completato questa fase, verifica che:

1. **Push automatico funziona**:
   ```bash
   git checkout -b test-push
   echo "test" > test.txt
   git add test.txt
   git commit -m "test: prova push"
   # La skill dovrebbe fare push automaticamente
   ```

2. **PR viene creata**:
   - Controlla che la PR esista su GitHub
   - Che il titolo sia `Cantiere: fase <N> — <titolo>`
   - Che il body contenga la descrizione

3. **Retry esponenziale funziona**:
   - Testa con connessione intermittente
   - Verifica che riprovi 4 volte con backoff

## 📝 Note per l'esecuzione

- La skill già legge FASI.md e esegue le fasi
- Devi aggiungere la logica di push/PR dopo il commit
- Usa i GitHub MCP tools per le operazioni su GitHub
- Mantieni i messaggi di feedback chiari e in italiano

## 🎯 Criterio di completamento

La fase è completata quando:
- ✅ Push automatico funziona (con retry)
- ✅ PR viene creata automaticamente
- ✅ PR viene aggiornata per fasi successive
- ✅ Tutti i messaggi sono in italiano
- ✅ Test manuali passano

---

**Prossima fase**: [Fase 2: Rinomina File Conclusi](./cantiere-fasi-markdown-fase-2.md)

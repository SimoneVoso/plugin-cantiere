# 🏗️ CANTIERE: Fasi Markdown

**Status**: 🔨 In esecuzione
**Fase corrente**: 1 di 5
**Data inizio**: 2026-08-20
**Ultimo aggiornamento**: 2026-08-20 - Documentazione creata
**Ultimo commit**: 69fd3a7

---

## 📋 Descrizione del cantiere

Implementazione del **nuovo sistema a fasi** per il plugin-cantiere:
- Sostituzione del sistema a capitoli con un sistema a fasi (1-5)
- Automatizzazione completa di push e creazione PR
- Documentazione e skill in italiano
- Struttura standardizzata e verificabile

## 🎯 Obiettivo finale

Avere un sistema di cantiere completamente funzionale dove:
- ✅ `/cantiere:apri-fasi` apre il cantiere con FASI.md
- ✅ `/cantiere:fase N` esegue una fase singola
- ✅ Push e PR sono automatici
- ✅ Ogni fase è una sessione nuova e isolata
- ✅ Documentazione completa in italiano

## 📊 Le 5 Fasi

| Fase | Titolo | Descrizione | Status |
|------|--------|-------------|--------|
| 1 | Automazione Push/PR | Implementare push automatico con retry e PR auto | 🔄 In corso |
| 2 | Rinomina File Conclusi | File fase auto-rinominati con `_conclusa` | ⏳ In attesa |
| 3 | Integrazione GitHub MCP | Usare GitHub MCP tools per PR | ⏳ In attesa |
| 4 | Testing Completo | Test su tutte le fasi e scenari | ⏳ In attesa |
| 5 | Documentazione Finale | README, esempi, guida utente | ⏳ In attesa |

---

## 🔗 Documenti delle Fasi

- [Fase 1: Automazione Push/PR](./cantiere-fasi-markdown-fase-1.md)
- [Fase 2: Rinomina File Conclusi](./cantiere-fasi-markdown-fase-2.md)
- [Fase 3: Integrazione GitHub MCP](./cantiere-fasi-markdown-fase-3.md)
- [Fase 4: Testing Completo](./cantiere-fasi-markdown-fase-4.md)
- [Fase 5: Documentazione Finale](./cantiere-fasi-markdown-fase-5.md)

---

## 📝 Note

- Ogni fase ha il suo documento dedicato
- Quando una fase è conclusa, il file viene rinominato con `_conclusa`
- La PR principale rimane aperta durante tutto il cantiere
- Tutti i commit hanno il prefisso `cantiere fase:`

## ✅ Fase Concluse

(Nessuna ancora)

---

**Apri la [Fase 1](./cantiere-fasi-markdown-fase-1.md) per iniziare!**

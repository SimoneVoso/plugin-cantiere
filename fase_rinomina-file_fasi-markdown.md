# Fase 2: Rinomina File Conclusi

**Cantiere**: Fasi Markdown  
**Fase**: 2 di 5  
**Status**: ⏳ In attesa  
**Prerequisiti**: Fase 1 completata

---

## 📌 Obiettivo

Implementare il **rinomina automatico** dei file di fase quando vengono conclusi:
- Quando fase completata → `cantiere-fasi-markdown-fase-N_conclusa.md`
- Aggiornare CANTIERE.md con le fasi concluse
- Aggiornare FASI.md con lo stato

## 📋 Checklist

- [ ] Implementare logica di rinomina file
- [ ] Aggiornare CANTIERE.md automaticamente
- [ ] Aggiornare FASI.md con il nuovo stato
- [ ] Testare rinomina per ogni fase
- [ ] Committa il rinomina nel commit della fase

## 🔧 File da modificare

| File | Cosa fare |
|------|-----------|
| `plugins/cantiere/skills/fase/SKILL.md` | Aggiungere logica rinomina dopo verifica |
| `cantiere-fasi-markdown-fase-N.md` | Auto-rinominato dopo completamento |
| `CANTIERE.md` | Aggiornato con stato fasi |
| `FASI.md` | Aggiornato con riga di stato |

## ✅ Verifica

Dopo aver completato questa fase, verifica che:

1. **File rinominato correttamente**:
   ```bash
   ls -la cantiere-fasi-markdown-fase-*
   # Dovrebbe mostrare: cantiere-fasi-markdown-fase-1_conclusa.md
   ```

2. **CANTIERE.md aggiornato**:
   - Fase 1 segnata come ✅ Conclusa
   - Timestamp aggiornato
   - Link funzionante a fase-1_conclusa.md

3. **FASI.md aggiornato**:
   - Riga di stato: "Fase corrente: 2 di 3"
   - Ultimo commit: SHA della fase 1

## 📝 Note per l'esecuzione

- La rinomina deve accadere **dopo** la verifica passata
- La rinomina deve essere committata nello stesso commit della fase
- Mantieni i link funzionanti nel CANTIERE.md
- Aggiorna sia il documento che i file YAML/config

## 🎯 Criterio di completamento

La fase è completata quando:
- ✅ File rinominato automaticamente con `_conclusa`
- ✅ CANTIERE.md riflette lo stato aggiornato
- ✅ FASI.md aggiornato con riga di stato
- ✅ Link nel CANTIERE.md rimangono funzionanti
- ✅ Rinomina è committata nel commit della fase

---

**Prossima fase**: [Fase 3: Integrazione GitHub MCP](./fase_integrazione-github_fasi-markdown.md)

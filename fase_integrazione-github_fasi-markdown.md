# Fase 3: Integrazione GitHub MCP

**Cantiere**: Fasi Markdown  
**Fase**: 3 di 5  
**Status**: ⏳ In attesa  
**Prerequisiti**: Fase 1-2 completate

---

## 📌 Obiettivo

Integrare completamente i **GitHub MCP tools** nella skill `/cantiere:fase` per operazioni robuste su PR:
- Usare `mcp__github__create_pull_request` per creare PR
- Usare `mcp__github__update_pull_request` per aggiornare PR
- Gestire i casi edge (PR già esiste, etc)
- Error handling completo

## 📋 Checklist

- [ ] Importare GitHub MCP tools nella skill
- [ ] Implementare creazione PR con body strutturato
- [ ] Implementare update PR per fasi successive
- [ ] Gestire errore PR già esiste
- [ ] Gestire errori di autenticazione
- [ ] Aggiungere retry logic per operazioni GitHub
- [ ] Testare tutti gli scenari

## 🔧 File da modificare

| File | Cosa fare |
|------|-----------|
| `plugins/cantiere/skills/fase/SKILL.md` | Refactor push/PR con MCP tools |
| Nessun nuovo file | Solo modifica SKILL.md |

## ✅ Verifica

Dopo aver completato questa fase, verifica che:

1. **PR creata con MCP tool**:
   ```bash
   # Controlla su GitHub che la PR esista
   # Che il title sia corretto
   # Che il body sia formattato bene
   ```

2. **PR aggiornata per fase 2**:
   - Il body della PR riflette il completamento della fase 2
   - Gli aggiornamenti sono incrementali (non riscritti da zero)

3. **Error handling funziona**:
   - Se PR esiste, aggiorna invece di creare
   - Se autenticazione fallisce, fallisce pulito
   - Se rate limit, riprova dopo attesa

## 📝 Note per l'esecuzione

- I GitHub MCP tools sono disponibili nel tool search
- Usa `mcp__github__create_pull_request` per la prima fase
- Usa `mcp__github__update_pull_request` per le fasi successive
- Mantieni il body della PR aggiornato ma leggibile

## 🎯 Criterio di completamento

La fase è completata quando:
- ✅ PR creata tramite MCP tool (non manualmente)
- ✅ PR aggiornata automaticamente per ogni fase
- ✅ Body della PR è ben formattato e chiaro
- ✅ Error handling per tutti gli scenari
- ✅ Nessun fallimento dovuto a GitHub

---

**Prossima fase**: [Fase 4: Testing Completo](./fase_testing-completo_fasi-markdown.md)

# Fase 4: Testing Completo

**Cantiere**: Fasi Markdown  
**Fase**: 4 di 5  
**Status**: ⏳ In attesa  
**Prerequisiti**: Fase 1-3 completate

---

## 📌 Obiettivo

Testare **completamente** il sistema a fasi in tutti gli scenari:
- Test positivo: tutto funziona
- Test fallimenti: push fallisce, PR fallisce, etc
- Test edge cases: fase 5 (ultima), interruzione, etc
- Test manuale: lanciare il sistema completo

## 📋 Checklist

- [ ] Test fase 1: push e PR creata
- [ ] Test fase 2: PR aggiornata, file rinominato
- [ ] Test fase 3: MCP tools usati correttamente
- [ ] Test fallimento push (simula rete flaky)
- [ ] Test fallimento PR (simula errore GitHub)
- [ ] Test ultima fase (fase 5): cleanup suggerito
- [ ] Test interruzione in mezzo a una fase
- [ ] Test skip di una fase (non consigliato)

## 🔧 File da testare

| Skill | Scenario |
|-------|----------|
| `/cantiere:apri-fasi` | Crea FASI.md correttamente |
| `/cantiere:fase 1` | Esegue, push, crea PR |
| `/cantiere:fase 2` | Esegue, push, aggiorna PR, rinomina file |
| `/cantiere:fase 3` | Esegue, push, aggiorna PR |
| `/cantiere:fase 4` | Esegue, push, aggiorna PR |
| `/cantiere:fase 5` | Esegue, push, aggiorna PR, chiude cantiere |

## ✅ Verifica

Dopo aver completato questa fase, verifica che:

1. **Test positivo completo**:
   ```
   Sessione 1: /cantiere:apri-fasi "Test Cantiere"
   Sessione 2: /cantiere:fase 1
   Sessione 3: /cantiere:fase 2
   Sessione 4: /cantiere:fase 3
   Sessione 5: /cantiere:fase 4
   Sessione 6: /cantiere:fase 5
   ```
   - Ogni sessione completa correttamente
   - PR aggiornata 5 volte
   - File rinominati con `_conclusa`
   - CANTIERE.md mostra ✅ su tutte le fasi

2. **Test fallimenti**:
   - Push fallisce → retry esponenziale funziona
   - PR fallisce → messaggio di errore chiaro
   - GitHub MCP tools fallisce → error handling pulito

3. **Test edge cases**:
   - Fase 5 suggerisce cleanup
   - CANTIERE.md è chiuso/concluso
   - File FASI.md è pronto per essere cancellato

## 📝 Note per l'esecuzione

- Testa in un branch di test separato
- Usa valori reali (non simulati) per GitHub
- Prendi screenshot dei risultati
- Documenta qualunque bug trovato

## 🎯 Criterio di completamento

La fase è completata quando:
- ✅ Test positivo: tutto da fase 1 a 5 funziona
- ✅ Test fallimenti: retry e error handling funzionano
- ✅ Test edge cases: ultima fase e cleanup funzionano
- ✅ PR ha 5 aggiornamenti (uno per fase)
- ✅ Nessun bug critico trovato

---

**Prossima fase**: [Fase 5: Documentazione Finale](./fase_documentazione_fasi-markdown.md)

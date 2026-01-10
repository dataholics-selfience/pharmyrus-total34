# ⚡ PHARMYRUS v29.9 - OTIMIZAÇÕES PARA ASPIRIN

---

## ❌ PROBLEMA: Aspirin Timeout

**Erro:** `SoftTimeLimitExceeded` após ~31 minutos

**Causa:**
- 210 BRs para enriquecer
- 42 batches × 5 BRs cada
- Timeout Celery: 55 minutos
- Tempo real: ~31 minutos só no INPI enrichment
- **Total excedeu 55 minutos!**

**Logs:**
```
Batch 6/42 (5 BRs)...
Batch 7/42 (5 BRs)...
...
Batch 20/42... ← ~21 min
...
❌ Error: Page.click: Timeout 10000ms exceeded
❌ SoftTimeLimitExceeded()
```

---

## ✅ SOLUÇÕES v29.9

### 1. **Aumentar Timeout Celery**

**ANTES:**
```python
task_time_limit=3600,  # 60 min
task_soft_time_limit=3300,  # 55 min
```

**DEPOIS:**
```python
task_time_limit=5400,  # v29.9: 90 min (era 60)
task_soft_time_limit=5100,  # v29.9: 85 min (era 55)
```

**Resultado:** +30 minutos de margem!

### 2. **Aumentar Batch Size INPI**

**ANTES:**
```python
BATCH_SIZE = 5  # 210 BRs = 42 batches
```

**DEPOIS:**
```python
BATCH_SIZE = 10  # v29.9: 210 BRs = 21 batches (METADE!)
```

**Resultado:**
- 42 batches → 21 batches
- ~42 minutos → ~21 minutos (METADE!)
- Overhead reduzido 50%

### 3. **Aumentar Timeouts INPI Clicks**

**ANTES:**
```python
await page.click(..., timeout=10000)  # 10 segundos
```

**DEPOIS:**
```python
await page.click(..., timeout=20000)  # v29.9: 20 segundos
```

**Resultado:** Menos erros de timeout em clicks!

---

## 📊 IMPACTO ESPERADO

### Aspirin (~210 BRs):

**v29.8:**
```
INPI Enrichment:
  - 42 batches × 5 BRs
  - ~31+ minutos
  - Timeout: 55 min
  - Status: ❌ TIMEOUT!
```

**v29.9:**
```
INPI Enrichment:
  - 21 batches × 10 BRs
  - ~15-20 minutos (METADE!)
  - Timeout: 85 min
  - Status: ✅ OK!
```

### Darolutamide (~20 BRs):

**Sem impacto** - continua rápido (~2-3 batches)

---

## 🔧 MUDANÇAS NO CÓDIGO

### `celery_app.py`

**Linha 39-40:**
```python
# ANTES:
task_time_limit=3600,  # 60 min
task_soft_time_limit=3300,  # 55 min

# DEPOIS:
task_time_limit=5400,  # v29.9: 90 min
task_soft_time_limit=5100,  # v29.9: 85 min
```

### `main.py`

**Linha ~1387:**
```python
# ANTES:
BATCH_SIZE = 5

# DEPOIS:
BATCH_SIZE = 10  # v29.9: Duplicado!
```

### `inpi_crawler.py`

**Múltiplas linhas:**
```python
# ANTES:
timeout=10000  # 12 ocorrências

# DEPOIS:
timeout=20000  # v29.9: Todas aumentadas!
```

---

## ✅ VALIDAÇÕES

### Aspirin deve completar:
```
📊 Processing 21 batches of 10 BRs each...  ← ERA 42!
Batch 1/21 (10 BRs)...
...
Batch 21/21 (10 BRs)...
✅ INPI Enrichment Complete: 210/210 BRs enriched

✅ Search completed in ~40-50 min  ← <85 min limite!
```

### Darolutamide não impactado:
```
📊 Processing 2 batches of 10 BRs each...
✅ Completa em ~10 min (como antes)
```

---

## 📝 LOGS ESPERADOS

```
🔄 Processing 21 batches of 10 BRs each...  ← ERA 42!

📦 Batch 1/21 (10 BRs): BR112..., BR112..., ...  ← 10 BRs!
📄 1/10: BR112...
📄 2/10: BR112...
...
📄 10/10: BR112...
✅ INPI: Got details for 10/10 BRs  ← ERA 5/5!

📦 Batch 2/21 (10 BRs)...
...

✅ INPI Enrichment Complete: 210/210 BRs enriched
⏱️  Tempo total: ~20 minutos  ← ERA 31+!

✅ Search completed for aspirin in 2850s (47.5 min)  ← <85 min!
```

---

## ⚠️ TRADE-OFFS

### Batch Size Maior:

**Pros:**
- ✅ Menos batches (42 → 21)
- ✅ Menos overhead de login
- ✅ ~50% mais rápido

**Cons:**
- ⚠️ Se 1 BR falhar, 9 outras esperam
- ⚠️ Batches individuais mais lentos

**Conclusão:** Vale a pena! Overhead de login é o gargalo.

### Timeout Maior:

**Pros:**
- ✅ Moléculas grandes não falham
- ✅ Mais margem de segurança

**Cons:**
- ⚠️ Usuário espera mais se der erro

**Conclusão:** Necessário para Aspirin!

---

## 🎯 OUTRAS OTIMIZAÇÕES FUTURAS

Se ainda houver problemas:

1. **Paralelizar INPI enrichment** (2-3 batches simultâneos)
2. **Cache de BRs** (não re-enriquecer)
3. **Skip BRs antigas** (>10 anos)
4. **Progressive results** (retornar parcial)

---

**Versão:** v29.9-ASPIRIN-FIX  
**Data:** 2026-01-10  
**Mudanças:** 3 arquivos  
**Objetivo:** Aspirin completar sem timeout  
**Tempo esperado:** ~40-50 min (era timeout aos 55)

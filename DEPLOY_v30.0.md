# 🚀 PHARMYRUS v30.0-COMPLETE - DEPLOYMENT GUIDE

---

## ✅ FEATURES v30.0

### Core Features:
- ✅ Google Patents BR Direct Search (v29.6+)
- ✅ INPI Re-login every 4 searches (v29.8)
- ✅ Brand Auto-detection (v29.8)
- ✅ Cortellis Audit on top of JSON (v29.8)
- ✅ Sources preserved in merge (v29.7+)
- ✅ 90-minute timeout for Aspirin (v29.9)
- ✅ Batch size 10 for INPI enrichment (v29.9)
- ✅ 20-second timeouts for INPI clicks (v29.9)

### Key Logs to Expect:
```
🏷️ Brand auto-detected: Nubeqa
✅ Novo BR DIRETO: BR112...
Google Direct: 7 BRs
📊 Sources to merge:
   → EPO: 18 BRs
   → INPI Direct: 3 BRs  
   → Google Direct: 7 BRs
✅ Merged: 25 unique BRs from 2 sources
   → EPO: 18 BRs
   → Google Patents Direct: 7 BRs
📊 CORTELLIS AUDIT: X/8 BRs found (XX%)
```

---

## 📦 DEPLOYMENT STEPS

### 1. GitHub Push
```bash
cd pharmyrus-total31-main
git add .
git commit -m "Deploy v30.0-COMPLETE: Google Patents BR + Optimizations"
git push origin main
```

### 2. Railway Deploy
Railway should **auto-deploy** from GitHub.

**OR manually:**
- Go to Railway dashboard
- Click "Deploy" on your service
- Wait for build to complete (~3-5 min)

### 3. Verify Deployment
```bash
# Check logs for version
curl https://your-railway-app.up.railway.app/health

# Or check Railway logs:
# Should show: "Pharmyrus v30.0-COMPLETE"
```

---

## 🔍 POST-DEPLOY VALIDATION

### Test with Darolutamide:
```json
POST /search
{
  "nome_molecula": "darolutamide",
  "paises_alvo": ["BR"],
  "incluir_wo": false
}
```

### Expected Results:

#### 1. Cortellis Audit (TOP of JSON):
```json
{
  "cortellis_audit": {
    "total_cortellis_brs": 8,
    "found": 6-8,
    "recall": 75-100,
    "rating": "MEDIUM" or "HIGH",
    "matched_brs": ["BR112...", ...],
    "missing_brs": [...]
  }
}
```

#### 2. Metadata:
```json
{
  "metadata": {
    "version": "Pharmyrus v30.0-COMPLETE",  ← CHECK!
    "brand_name": "Nubeqa",  ← NOT "ODM-201"!
    ...
  }
}
```

#### 3. By Source:
```json
{
  "patent_discovery": {
    "summary": {
      "by_source": {
        "EPO": 18-20,
        "INPI": 20-23,
        "Google Patents": 6-8,  ← > 0!
        "WIPO": 2
      }
    }
  }
}
```

#### 4. Logs Should Show:
```
🏷️ Brand auto-detected: Nubeqa
✅ Novo BR DIRETO: BR112021014969A2
✅ Novo BR DIRETO: BR112020023136A2
...
Google Direct: 7 BRs
📊 Sources to merge:
   → Google Direct: 7 BRs
✅ Merged: 25 unique BRs from 2 sources
   → Google Patents Direct: 7 BRs
📊 CORTELLIS AUDIT: 6-8/8 BRs
🔄 Query #5: RE-LOGIN preventivo (a cada 4)
```

---

## ⚠️ COMMON ISSUES

### Issue 1: `Google Patents: 0`

**Cause:** Old version (v31.0) still deployed

**Fix:**
```bash
# Force rebuild on Railway:
1. Go to Settings → Redeploy
2. Or push a dummy commit:
   git commit --allow-empty -m "Force rebuild"
   git push
```

### Issue 2: Brand = "ODM-201" (dev code)

**Cause:** Old brand detection logic

**Fix:** Same as Issue 1 - redeploy v30.0

### Issue 3: Timeout on Aspirin

**Cause:** Old timeout settings

**Check:**
```python
# celery_app.py should have:
task_soft_time_limit=5100,  # 85 min
```

**Fix:** Redeploy v30.0

### Issue 4: INPI re-login errors

**Logs show:**
```
Query 8/15: ...
❌ Error searching: ...
```

**Fix:** v30.0 has re-login every 4 (was 5/8)

---

## 📊 PERFORMANCE BENCHMARKS

### Darolutamide (~20 BRs):
- Time: ~10-15 minutes
- BRs found: 20-25
- Cortellis recall: 75-100%
- Google BRs: 6-8

### Aspirin (~210 BRs):
- Time: ~40-60 minutes (< 85 min limit)
- BRs found: 200-220
- INPI batches: 21 (was 42)

---

## 🎯 SUCCESS CRITERIA

✅ Version = "v30.0-COMPLETE"
✅ Brand = "Nubeqa" (for Darolutamide)
✅ Google Patents > 0 in by_source
✅ Cortellis audit on top of JSON
✅ Cortellis recall > 50%
✅ No timeout errors on Aspirin
✅ INPI re-login at queries 5, 9, 13

---

## 🚨 ROLLBACK PROCEDURE

If v30.0 has issues:

```bash
# Revert to last working version
git revert HEAD
git push

# Or checkout specific version:
git checkout <commit-hash>
git push -f
```

---

## 📝 VERSION HISTORY

- **v30.0-COMPLETE**: All features integrated
- **v29.9**: Aspirin timeout fixes
- **v29.8**: Brand detection + Cortellis audit
- **v29.7**: Google BRs in merge
- **v29.6**: Google Patents BR search
- **v31.0**: Old version (pre-Google BR)

---

**READY TO DEPLOY!** 🚀

Check Railway logs after deploy to confirm v30.0-COMPLETE is running.

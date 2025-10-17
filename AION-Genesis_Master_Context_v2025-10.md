# AION‑Genesis — Master Context (v2025-10)
**Fecha:** 2025-10-17  
**Estado:** Piloto cognitivo aislado (AION‑Genesis 4.0) — **Invariante #0** vigente.
**Propósito:** Contexto maestro para diseño, pruebas y evolución del meta‑agente cognitivo **AION‑Genesis**, separado 100% de CAC 2.0 para observar, aprender y mejorar sin afectar el entorno operativo.

---

## 0) Invariante #0 (AION‑Genesis Invariant)
> **Aislamiento total de CAC 2.0.**  
> Sin compartir bases, canales, servicios ni lógica. Observa sin tocar. Todo despliegue de AION‑Genesis es *sandbox* y no interfiere con el stack 2.x.

---

## 1) Objetivo y alcance (v0.1)
- **Meta‑agente IA‑nativo** que coordina sub‑agentes (visual, lingüístico, contextual, ético/Police, aprendizaje).  
- **Ciclo cognitivo completo:** evento → percepción → coherencia → decisión → Police → ejecución → registro → aprendizaje → adaptación.  
- **Latencia objetivo:** p50 ≤ 300 ms, p95 ≤ 400 ms.  
- **Reglas duras/Police** siempre por encima de cualquier heurística o aprendizaje.

---

## 2) Arquitectura mínima (v0.1)
- **Entradas:** eventos mínimos desde Home Assistant (*HA*) o simulador (un evento por sensor).  
- **Motor cognitivo:** calcula embeddings y coherencia multimodal (VT/VC/TC), estima riesgo y bienestar, propone `policy_vector`.  
- **Police:** veredictos `allow|deny|review` según reglas duras.  
- **Ejecución:** si *allow*, acciones hacia HA/Vocalis (en sandbox, modo virtual).  
- **Memoria:** registro **NDJSON** (“AIONResult v0.1”) y estado condensado `learning_state.json`.  
- **Aprendizaje incremental:** worker que resume experiencia y sugiere ajustes (sin hardcode).

**Diagrama conceptual (texto):**
```
① Evento (HA/sim) → ② Percepción (visual/linguistic/context + embeddings)
→ ③ Coherencia (VT, VC, TC, overall) → ④ Decisión (risk, wellbeing, policy_vector)
→ ⑤ Police (allow/deny/review) → ⑥ Ejecución (HA/Vocalis)
→ ⑦ Registro (NDJSON) → ⑧ Learning worker (state) → ⑨ Adaptación suave
```

---

## 3) Protocolo de evento mínimo (ingesta)
```json
{
  "event": {
    "id": "UUID",
    "source": "home_assistant",
    "sensor": "binary_sensor.puerta_entrada",
    "zone": "perimeter",
    "home_mode": "night"
  },
  "payload": {
    "identity": { "status": "unknown", "confidence": 0.0 },
    "emotion":  { "label": "neutral", "confidence": 0.5 }
  }
}
```
- `zone`: `perimeter|entry|indoor|garage|living|bedroom`  
- `home_mode`: `day|night|night_armed|away`  
- `identity_bucket` derivado: `unknown|known_weak|known_strong`

---

## 4) Formato de registro — **AIONResult v0.1** (NDJSON, append‑only)
**Ruta:** `{AION_RESULTS_DIR}/YYYY-MM-DD.ndjson`

Campos clave por línea (ejemplo abreviado):
```json
{
  "schema_version": "aion_result/0.1",
  "ts": 1760888405.123,
  "event": {"id":"...","source":"home_assistant","sensor":"...","zone":"perimeter","home_mode":"night"},
  "modalities": {
    "visual":    {"emb":[...],"conf":0.86,"labels":{"person":0.99},"emotion":{"sad":0.74}},
    "linguistic":{"emb":[...],"conf":0.72,"text":"todo bien"},
    "context":   {"emb":[...],"conf":0.90,"features":{"hour":"23:41","presence":true}}
  },
  "decision": {
    "policy_vector":{"light.warm":0.6,"notify.brief":0.3},
    "risk_level":"high",
    "wellbeing_delta":-0.15,
    "explanation_short":"Desconocido en perímetro de noche; luz exterior ON."
  },
  "metrics": {"coherence":{"vt":0.41,"vc":0.78,"tc":0.52,"overall":0.57},"latency_ms":238},
  "credit": {"signal":-0.2,"reason":"Police=deny; ruido innecesario evitado"},
  "police": {"verdict":"deny","reason":"Regla: unknown@perimeter@night → strict"},
  "trace": {"models":{"vision":"yolov8n","nlp":"gpt-mini","embed":"e5-large"},"aion_genesis_id":"GEN-2025-10-α3"}
}
```
**Notas:**
- `emb` (embeddings) normalizados L2; omitir modalidad no disponible.  
- `policy_vector` escaso (acciones con peso > 0.1).  
- NDJSON → fácil de auditar y alimentar aprendizaje.

---

## 5) Métrica de coherencia (v0.1)
- Similitud coseno por pares de embeddings → reescalado a [0,1].  
  - **VT**: visual↔texto, **VC**: visual↔contexto, **TC**: texto↔contexto.  
- Agregado: `overall = clamp01(w_vt*VT + w_vc*VC + w_tc*TC)`  
  - ENV por defecto: `COH_W_VT=0.40`, `COH_W_VC=0.40`, `COH_W_TC=0.20`  
  - Umbrales: `COH_ALERT_THR=0.35`, `COH_STRICT_THR=0.25`

---

## 6) Aprendizaje incremental (worker v0.1)
**Entradas:** NDJSON de últimos *N* días (`LEARN_LOOKBACK_DAYS`) o *M* eventos (`LEARN_MAX_EVENTS`).  
**Salidas:** `{AION_STATE_DIR}/learning_state.json` (≤ 50 KB) y (opcional) series `trend_*.json`.

`learning_state.json` (ejemplo mínimo):
```json
{
  "schema_version":"aion_learning/0.1",
  "window":{"days":7,"events":18342},
  "globals":{"avg_coherence":0.71,"avg_credit":0.08,"avg_latency_ms":212},
  "by_context":[{"key":"perimeter@night","avg_coh":0.62,"avg_credit":0.04,"allow":0.11,"deny":0.77,"review":0.12}],
  "policy":{"top_effective":[["light.warm",0.19]],"top_ineffective":[["siren",-0.27]],"pattern_fail":["unknown@perimeter@night"]},
  "risk":{"dist":{"low":0.28,"medium":0.41,"high":0.24,"critical":0.07},"drift_24h":{"critical":0.02}},
  "coherence":{"avg_overall":0.71,"low_conflict_rate":0.18,"suggest":{"COH_ALERT_THR":"+0.03","COH_STRICT_THR":"+0.01"}},
  "updated_at":"2025-10-16T22:10:00Z"
}
```
**Uso:** el CAC lee este estado para *sesgar suavemente* (prioriza top_effective, endurece pattern_fail). Guardrails/Police prevalecen.

---

## 7) Police v0.1 (reglas duras mínimas)
- `unknown@perimeter@night(_armed)` → **deny** `siren/camera.record/listen.area`; permitir solo `light.alert|notify`.  
- `unknown@indoor@night(_armed)` → **deny** + **review** siempre.  
- `coherence.overall < COH_ALERT_THR` → **review**.  
- `coherence.overall < COH_STRICT_THR` & `night(_armed)` → **deny**.  
- `known_strong@perimeter@day` → **allow** `tts.greet|light.warm`.  
- `known_weak@perimeter@night` → **review`.  
- `risk=critical & coherence>0.6` → **allow** plan crítico (según privacidad).  
- `wellbeing_delta < -0.5` → **review**.  
- `Police.override_active=true` → **deny** todo (emergencia).

---

## 8) Canon de acciones (policy_vector v0.1)
| Acción | Propósito | Canal | Costo |
|-------|-----------|-------|-------|
| `light.warm` | Luz de cortesía | HA | Bajo |
| `light.alert` | Luz disuasiva | HA | Medio |
| `notify.brief` | Notificación breve | Vocalis/HA | Bajo |
| `notify.urgent` | Notificación prioritaria | Vocalis/HA | Medio |
| `siren` | Alarma sonora | HA | Alto |
| `camera.record` | Captura/Grabación | HA/Watcher | Medio |
| `listen.area` | Micrófono de zona | HA/local | Alto |
| `tts.greet` | Saludo empático | Vocalis | Bajo |

> El “costo” pondera `credit.signal`. Acciones caras requieren mayor justificación.

---

## 9) Taxonomía de contexto (v0.1)
- `zone`: `perimeter|entry|indoor|garage|living|bedroom`  
- `home_mode`: `day|night|night_armed|away` (si alarma activa y noche → `night_armed`)  
- `identity_bucket`: `unknown|known_weak|known_strong` (confianza >0.8 → strong; 0.5–0.8 → weak)

---

## 10) Interfaces sandbox (API mínima)
- `POST /ingest/event` → ingesta de evento (HA/sim).  
- `GET  /aion/learning/state` → snapshot del aprendizaje.  
- `GET  /aion/health` → estado básico del agente.  
- `POST /aion/feedback` → (futuro) feedback usuario para `credit.signal`.

---

## 11) Observabilidad mínima
- Contadores: `events_total`, `allow|deny|review`.  
- Gauges: `avg_latency_ms`, `avg_coherence_overall`, `avg_credit_signal`, `conflict_rate`.  
- Dump periódico en `{AION_STATE_DIR}/../metrics.json` o MQTT `aion/metrics`.

---

## 12) Privacidad & trazabilidad (ligero)
- PII: nombres → hash SHA1 (`Jorge Day` → `jd_4a2f…`).  
- Video/Audio: solo bajo `camera.record` explícito.  
- Logs: sin textos íntegros ni voz; solo UUID/zonas.  
- Integridad: hash SHA256 por archivo NDJSON diario (opcional).  
- Retención: 30 días (rotación diaria + compresión semanal).

---

## 13) ENV Seed inicial (no hardcode)
```
COH_W_VT=0.40
COH_W_VC=0.40
COH_W_TC=0.20
COH_ALERT_THR=0.35
COH_STRICT_THR=0.25
CRED_A1=0.7
CRED_A2=0.2
CRED_A3=0.1
LEARN_LOOKBACK_DAYS=7
LEARN_MAX_EVENTS=20000
LEARN_BATCH_EVENTS=500
MIN_USAGE=25
FAIL_THR=-0.10
DRIFT_THR=0.08
AION_RESULTS_DIR=/data/aion/results
AION_STATE_DIR=/data/aion/state
AION_LEARNING_TOPIC=aion/learning/state
```

---

## 14) Casos de uso y pruebas (resumen)
Ver anexo: **AION‑Genesis_Casos_de_Uso_y_Pruebas_v0.1.md** (S1–S12).  
Criterios globales (DoD): p95≤400 ms; 0 falsos críticos; coherencia media≥0.60; Police cumple matriz; NDJSON + learning_state generados.

---

## 15) Anexos oficiales
- **AION‑Genesis_Design_v0.1_Cierre_Arquitectonico.md** — diseño y 10 puntos.  
- **AION‑Genesis_Diccionario_de_Datos_v0.1.md** — términos y definiciones.  
- **AION‑Genesis_Casos_de_Uso_y_Pruebas_v0.1.md** — escenarios y DoD.

---

### Notas finales
- Este *Master Context* es la fuente de verdad para el piloto v0.1.  
- Cualquier cambio debe mantener el **Invariante #0** y evitar hardcode/configs excesivos.  
- El objetivo del piloto es **aprender del hogar** sin riesgo: refinar coherencia, Police y eficacia de acciones.

# AION-Genesis — Master Context (v2025-10)

**Fecha:** 2025-10-17  
**Estado:** Piloto cognitivo aislado (AION-Genesis 4.0) — **Invariante #0** vigente.  
**Propósito:** Contexto maestro para diseño, pruebas y evolución del meta-agente cognitivo **AION-Genesis**, separado 100% de CAC 2.0 para observar, aprender y mejorar sin afectar el entorno operativo.

---

## 0) Invariante #0 (AION-Genesis Invariant)
> **Aislamiento total de CAC 2.0.**  
> Sin compartir bases, canales, servicios ni lógica. Observa sin tocar. Todo despliegue de AION-Genesis es *sandbox* y no interfiere con el stack 2.x.

---

## 1) Objetivo y alcance (v0.1)
- **Meta-agente IA-nativo** que coordina sub-agentes (visual, lingüístico, contextual, ético/Police, aprendizaje).  
- **Ciclo cognitivo completo:** evento → percepción → coherencia → decisión → Police → ejecución → registro → aprendizaje → adaptación.  
- **Latencia objetivo:** p50 ≤ 300 ms, p95 ≤ 400 ms.  
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
    "device_class": "door",
    "state": "on",
    "prev_state": "off",
    "zone": "perimeter",
    "home_mode": "night"
  },
  "ts": 1760888405.123
}
```
- `zone`: `perimeter|entry|indoor|garage|living|bedroom`  
- `home_mode`: `day|night|night_armed|away`  
- **Regla:** el contrato es **sensor-centrado** (HA).  
  Cualquier identidad/emoción/texto proviene de sub-agentes (Watcher, Vocalis, etc.)  
  y se refleja **solo** en `modalities` del **AIONResult**, **no** en la ingesta.

---

## 4) Formato de registro — **AIONResult v0.1** (NDJSON, append-only)
**Ruta:** `{AION_RESULTS_DIR}/YYYY-MM-DD.ndjson`

Campos clave por línea (ejemplo abreviado):
```json
{
  "schema_version": "aion_result/0.1",
  "ts": 1760888405.123,
  "event": {
    "id":"...",
    "source":"home_assistant",
    "sensor":"...",
    "device_class":"door",
    "state":"on",
    "prev_state":"off",
    "zone":"perimeter",
    "home_mode":"night"
  },
  "modalities": {
    "visual":{"conf":0.86,"labels":{"person":0.99},"emotion":{"sad":0.74}},
    "linguistic":{"conf":0.72,"text":"todo bien"},
    "context":{"conf":0.90,"features":{"hour":"23:41","presence":true}}
  },
  "decision": {
    "policy_vector":{"light.warm":0.6,"notify.brief":0.3},
    "risk_level":"high",
    "wellbeing_delta":-0.15,
    "explanation_short":"Desconocido en perímetro de noche; luz exterior ON."
  },
  "metrics":{"coherence":{"vt":0.41,"vc":0.78,"tc":0.52,"overall":0.57},"latency_ms":238},
  "credit":{"signal":-0.2,"reason":"Police=deny; ruido innecesario evitado"},
  "police":{"verdict":"deny","reason":"Regla: unknown@perimeter@night → strict"},
  "trace":{"models":{"vision":"yolov8n","nlp":"gpt-mini"},"aion_genesis_id":"GEN-2025-10-α3"}
}
```
**Notas:** embeddings L2; `policy_vector` escaso (peso >0.1); modalidades ausentes se omiten.

---

## 5) Métrica de coherencia (v0.1)
- Coseno por pares → reescalado [0,1].  
  - **VT:** visual↔texto · **VC:** visual↔contexto · **TC:** texto↔contexto  
- `overall = clamp01(w_vt*VT + w_vc*VC + w_tc*TC)`  
  - ENV: `COH_W_VT=0.40`, `COH_W_VC=0.40`, `COH_W_TC=0.20`  
  - Umbrales: `COH_ALERT_THR=0.35`, `COH_STRICT_THR=0.25`

---

## 6) Aprendizaje incremental (worker v0.1)
**Entradas:** NDJSON recientes (`LEARN_LOOKBACK_DAYS` o `LEARN_MAX_EVENTS`).  
**Salida:** `{AION_STATE_DIR}/learning_state.json` (≤50 KB) y series `trend_*.json`.

Ejemplo:
```json
{
  "schema_version":"aion_learning/0.1",
  "window":{"days":7,"events":18342},
  "globals":{"avg_coherence":0.71,"avg_credit":0.08,"avg_latency_ms":212},
  "risk":{"dist":{"low":0.28,"medium":0.41,"high":0.24,"critical":0.07}},
  "updated_at":"2025-10-16T22:10:00Z"
}
```
Uso: sesgo suave (prioriza top_effective, endurece pattern_fail).  
**Police prevalece.**

---

## 7) Police v0.1 (reglas duras mínimas)
- `unknown@perimeter@night(_armed)` → **deny** intrusivos; permitir `light.alert|notify`.  
- `unknown@indoor@night(_armed)` → **deny** + **review**.  
- `coherence.overall < COH_ALERT_THR` → review.  
- `coherence.overall < COH_STRICT_THR & night(_armed)` → deny.  
- `known_strong@perimeter@day` → allow `tts.greet|light.warm`.  
- `Police.override_active=true` → deny todo.

---

## 8) Canon de acciones (policy_vector v0.1)
| Acción | Propósito | Canal | Costo |
|--------|------------|-------|-------|
| `light.warm` | Luz de cortesía | HA | Bajo |
| `light.alert` | Luz disuasiva | HA | Medio |
| `notify.brief` | Notificación breve | Vocalis/HA | Bajo |
| `notify.urgent` | Notificación prioritaria | Vocalis/HA | Medio |
| `siren` | Alarma sonora | HA | Alto |
| `camera.record` | Captura/Grabación | HA/Watcher | Medio |
| `listen.area` | Micrófono de zona | HA/local | Alto |
| `tts.greet` | Saludo empático | Vocalis | Bajo |

> El costo pondera `credit.signal`: acciones caras exigen mayor justificación.

---

## 9) Taxonomía de contexto (v0.1)
- `zone`: `perimeter|entry|indoor|garage|living|bedroom`  
- `home_mode`: `day|night|night_armed|away`  
- `identity_bucket`: `unknown|known_weak|known_strong` (solo sub-agentes)

---

## 10) Interfaces sandbox (API mínima)
- `POST /ingest/event` → ingesta (contrato §3).  
- `GET  /aion/learning/state` → snapshot de aprendizaje.  
- `GET  /aion/health` → estado básico.  
- `POST /aion/feedback` → feedback usuario.

---

## 11) Observabilidad mínima
- Contadores: `events_total`, `allow|deny|review`.  
- Gauges: `avg_latency_ms`, `avg_coherence_overall`, `avg_credit_signal`, `conflict_rate`.  
- Dump: `{AION_STATE_DIR}/../metrics.json` o MQTT `aion/metrics`.

---

## 12) Privacidad & trazabilidad
PII → hash SHA1; video/audio solo bajo `camera.record`; logs sin voz ni texto crudo; retención 30 días.

---

## 13) ENV Seed inicial
```
COH_W_VT=0.40
COH_W_VC=0.40
COH_W_TC=0.20
COH_ALERT_THR=0.35
COH_STRICT_THR=0.25
LEARN_LOOKBACK_DAYS=7
LEARN_MAX_EVENTS=20000
LEARN_BATCH_EVENTS=500
AION_RESULTS_DIR=/data/aion/results
AION_STATE_DIR=/data/aion/state
```

---

## 14) Casos de uso y pruebas (resumen)
Ver **AION-Genesis_Casos_de_Uso_y_Pruebas_v0.1.md** (S1–S12 + M1–M4).  
Criterios globales: p95≤400 ms; 0 falsos críticos; coherencia≥0.60.

---

## 15) Anexos oficiales
- **AION-Genesis_Design_v0.1_Cierre_Arquitectonico.md**  
- **AION-Genesis_Diccionario_de_Datos_v0.1.md**  
- **AION-Genesis_Casos_de_Uso_y_Pruebas_v0.1.md**

---

### Registro de cambios v2025-10-17
- Contrato de ingesta actualizado a formato **sensor-centrado**.  
- Eliminación de `payload.*` del esquema de entrada.  
- Alineación completa con diseño arquitectónico v0.1.  
- Coherencia revisada en terminología, anexos y métricas.  
- Conservación de estructura y secciones originales.


# AION-Genesis — Casos de Uso y Plan de Pruebas v0.1
**Fecha:** 2025-10-17  
**Propósito:** Validar el diseño arquitectónico v0.1 de AION‑Genesis en entorno *sandbox* mediante escenarios controlados (S1–S12), con criterios de aceptación, métricas y trazabilidad.  
**Relación:** Complementa *AION‑Genesis_Master_Context*, *Diccionario de Datos v0.1* y *Cierre Arquitectónico v0.1* (Invariante #0 respetado).

---

## 0) Alcance y objetivos
- Verificar ciclo cognitivo completo: **evento → percepción → coherencia → decisión → Police → ejecución → registro → aprendizaje**.
- Asegurar **no hardcode**, uso de **ENV seeds** y registros **NDJSON** con **learning_state.json**.
- Criterios globales de éxito del piloto:
  - Latencia **p50 ≤ 300 ms** y **p95 ≤ 400 ms**.
  - **0 falsos positivos críticos** (sirena/acciones intrusivas en contexto no validado).
  - **Coherencia promedio ≥ 0.60**.
  - Police cumple matriz S1–S12 (allow/deny/review).
  - Se genera **NDJSON** por evento y **learning_state.json** por ventana.

---

## 1) Entorno de prueba (sandbox)
- **Entradas:** eventos simulados de Home Assistant (curl) y/o reproductor de secuencias.
- **Salidas:** acciones virtuales (HA/Vocalis), registros NDJSON, metrics.json y learning_state.json.
- **Variables ENV clave:** usar las del *ENV Seed Inicial* (ver documento Cierre Arquitectónico v0.1).
- **Carpetas:** `/data/aion/results/`, `/data/aion/state/`, `/data/aion/metrics.json`.
- **Aislamiento total** de CAC 2.0 (Invariante #0).

---

## 2) Contextos canónicos (referencia)
- `zone`: `perimeter`, `entry`, `indoor`, `garage`, `living`, `bedroom`
- `home_mode`: `day`, `night`, `night_armed`, `away`
- `identity_bucket`: `unknown`, `known_weak`, `known_strong`

---

## 3) Matriz de escenarios (S1–S12) — resumen
| ID | Escenario | Expectativa | DoD (Definition of Done) |
|----|-----------|-------------|---------------------------|
| S1 | Conocido día/perímetro | `tts.greet` + `light.warm` | Police=allow, credit>0, coh≥0.6, lat≤400ms |
| S2 | Desconocido noche/perímetro | Solo luz disuasiva | Police=deny acciones intrusivas; risk=high |
| S3 | Desconocido noche/interior | Review | Police=review; no `siren/camera/voice` |
| S4 | Baja coherencia (visión vs texto) | Review | `coh<0.35` → review |
| S5 | Alta coherencia noche/conocido fuerte | Allow | `allow` + credit>0.1 |
| S6 | Modo away + movimiento interior | Critical | `risk=critical`, Police revisa coherencia, deny si `coh<strict` |
| S7 | Falla de cámara (sin visual) | Degradación segura | `coh` calculable con `context`; ningún intrusivo |
| S8 | Ruido de sensor (rebote 10s) | Anti‑replay | Deduplicación natural; 1 registro/decisión |
| S9 | Emoción negativa fuerte en conocido | Empatía + luz | `notify.brief` o `tts.greet` suave; no sirena |
| S10 | Intrusión confirmada coherente | Plan crítico | `allow` plan crítico; evidencia `camera.record` si Police lo permite |
| S11 | Drift de riesgo 24h | Aprendizaje | `learning_state.json` sugiere ajuste; métricas reflejan cambio |
| S12 | Usuario feedback negativo | Refuerzo | `credit.signal` ajusta eficacia de acción usada |

---

## 4) Casos por escenario (detallados)

### S1 — Conocido día / perímetro
**Setup:** `identity_bucket=known_strong`, `zone=perimeter`, `home_mode=day`  
**Input (ejemplo curl):**
```bash
curl -s -X POST http://localhost:8000/ingest/event -H "Content-Type: application/json" -d '{
  "event": {"id":"S1_001","source":"simulator","sensor":"camera.entrada_principal","zone":"perimeter","home_mode":"day"},
  "payload": {"identity":{"status":"known","confidence":0.93,"name":"Jorge Day"}, "emotion":{"label":"neutral","confidence":0.72}} }'
```
**Expected:** `policy_vector` contiene `tts.greet` y/o `light.warm`; `Police=allow`; `credit>0`.  
**DoD:** NDJSON con `coherence.overall≥0.6`, latencia≤400ms; metrics.json actualizado.

---

### S2 — Desconocido noche / perímetro
**Setup:** `unknown`, `perimeter`, `night`  
**Expected:** `light.alert` permitido; **deny** para `siren`, `camera.record`, `listen.area`.  
**DoD:** `risk=high`; `Police.reason` refleja regla estricta nocturna; `credit` neutro o levemente positivo.

---

### S3 — Desconocido noche / interior
**Setup:** `unknown`, `indoor`, `night_armed`  
**Expected:** `Police=review`; ninguna acción intrusiva.  
**DoD:** NDJSON registra `review`; coherencia evaluada; latencia≤400ms.

---

### S4 — Baja coherencia (visión dice “asustado”, texto dice “todo bien”)
**Setup:** `visual.emotion=anger/sad 0.8`, `linguistic.text="todo bien"`; `home_mode=day`  
**Expected:** `coherence.overall<0.35` → `Police=review`.  
**DoD:** Registro de `coherence.vt` bajo; explicación clara en `explanation_short`.

---

### S5 — Alta coherencia noche / conocido fuerte
**Setup:** `known_strong`, `perimeter`, `night`; `visual.emotion=happy 0.7`; sin contradicciones  
**Expected:** `allow` + `tts.greet` y `light.warm`; `credit>0.1`.  
**DoD:** NDJSON y metrics.json reflejan éxito y baja latencia.

---

### S6 — Modo away + movimiento interior
**Setup:** `home_mode=away`, `zone=indoor`, sensor presencia activo  
**Expected:** `risk=critical`; si `coh≥0.6`, `Police` puede permitir plan crítico; si `<strict`, `deny`.  
**DoD:** Acción se ajusta a coherencia; registro completo con `policy_vector` y `Police.reason`.

---

### S7 — Falla de cámara (sin modalidad visual)
**Setup:** `visual=null`, `context.presence=true`, `night`  
**Expected:** degradar a decisiones de **bajo costo** (`notify.brief`, `light.alert` opcional); no intrusivos.  
**DoD:** NDJSON omite `visual`; coherencia calculable con `tc`; Police prudente.

---

### S8 — Rebote de sensor (10s)
**Setup:** 5 eventos idénticos en 10s (`perimeter@night`)  
**Expected:** una sola decisión efectiva; los demás se consideran duplicados por ventana de tiempo del sandbox.  
**DoD:** 1 entrada relevante en NDJSON (resto `ignored=true` si aplica) y contadores consistentes.

---

### S9 — Conocido con emoción negativa fuerte
**Setup:** `known_strong@entry@day`, `emotion.sad 0.85`  
**Expected:** `notify.brief` empática o `tts.greet` suave; evitar `light.alert`/`siren`.  
**DoD:** `credit` no negativo; `wellbeing_delta` levemente positivo.

---

### S10 — Intrusión confirmada (coherente)
**Setup:** `unknown@indoor@night_armed`, `presence=true`, `coh≥0.6`  
**Expected:** plan crítico **permitido** por Police; opcional `camera.record` si la política de privacidad lo permite.  
**DoD:** NDJSON con `risk=critical`, lat≤400ms; evidencia si procede.

---

### S11 — Drift de riesgo (24h)
**Setup:** reproducir serie con aumento del 5–10% de eventos `high/critical` en 24h.  
**Expected:** `learning_state.json` refleja `risk.drift_24h` y `suggest` de ajustes prudentes.  
**DoD:** Worker ejecuta y persiste snapshot; métricas agregadas muestran cambio.

---

### S12 — Feedback negativo del usuario
**Setup:** tras acción `light.alert`, enviar feedback −1 (`/aion/feedback`).  
**Expected:** `credit.signal` ajustado en NDJSON; en ventana siguiente, eficacia de `light.alert` baja.  
**DoD:** learning_state muestra `top_ineffective` actualizado; recomendación visible.

---

## 5) Procedimiento general de prueba
1. **Preparación**: setear ENV; limpiar carpetas `/results` y `/state` (opcional, backup).  
2. **Ejecución**: inyectar eventos por escenario S1–S12 (scripts o curl).  
3. **Verificación**: revisar **NDJSON**, **metrics.json** y **learning_state.json**.  
4. **Métricas**: capturar latencias p50/p95, tasas allow/deny/review, coherencia media, crédito medio.  
5. **Cierre**: consolidar tabla de resultados y observaciones.

---

## 6) Registro mínimo por escenario (checklist)
- [ ] NDJSON contiene `event`, `modalities`, `decision`, `metrics.coherence`, `police`, `credit`, `trace`.  
- [ ] `policy_vector` solo con acciones canónicas (v0.1).  
- [ ] `police.verdict` y `reason` consistentes con reglas duras.  
- [ ] `latency_ms` ≤ 400 ms (p95).  
- [ ] `metrics.json` actualizado (contadores y promedios).  
- [ ] `learning_state.json` actualizado (cuando aplique).

---

## 7) Trazabilidad (requisitos ↔ pruebas)
| Requisito | Pruebas |
|-----------|---------|
| Aislamiento (Invariante #0) | Todos los escenarios en sandbox |
| Coherencia operativa | S4, S5, S7 |
| Police (reglas duras) | S2, S3, S6, S10 |
| Aprendizaje incremental | S11, S12 |
| Observabilidad | Verificación de metrics.json |
| Privacidad | S10 (condicionar `camera.record`) |

---

## 8) Anexos — ejemplos de payload y utilidades

### 8.1 Evento genérico (plantilla)
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
    "identity": {"status":"unknown","confidence":0.0},
    "emotion": {"label":"neutral","confidence":0.5}
  }
}
```

### 8.2 Comandos de apoyo
- Rotación manual de resultados (si se desea archivar rápido).
- Script de inyección por lotes (S8, S11).  
*(Dejar implementaciones para la fase de prototipado.)*

---

### Notas finales
- Este plan es **iterativo**: las tasas objetivo y umbrales pueden ajustarse tras el primer ciclo de aprendizaje.  
- Documentar *post‑mortems* breves para cualquier incumplimiento de DoD, alimentando mejoras en Police o coherencia.

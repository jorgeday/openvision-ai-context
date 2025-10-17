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
  - **0 falsos positivos críticos**.
  - **Coherencia promedio ≥ 0.60**.
  - Police cumple matriz S1–S12 (allow/deny/review).
  - Se genera **NDJSON** por evento y **learning_state.json** por ventana.

---

## 1) Entorno de prueba (sandbox)
- **Entradas:** eventos simulados de Home Assistant (curl) y/o reproductor de secuencias.
- **Salidas:** acciones virtuales (HA/Vocalis), registros NDJSON, metrics.json y learning_state.json.
- **Variables ENV:** usar las del *ENV Seed Inicial* (ver Cierre Arquitectónico v0.1).
- **Carpetas:** `/data/aion/results/`, `/data/aion/state/`, `/data/aion/metrics.json`.
- **Aislamiento total** de CAC 2.0 (Invariante #0).

---

## 2) Contextos canónicos
- `zone`: `perimeter`, `entry`, `indoor`, `garage`, `living`, `bedroom`
- `home_mode`: `day`, `night`, `night_armed`, `away`

---

## 3) Matriz de escenarios (S1–S12)
| ID | Escenario | Expectativa | DoD |
|----|-----------|-------------|-----|
| S1 | Conocido día/perímetro | `tts.greet` + `light.warm` | Police=allow, credit>0, coh≥0.6, lat≤400ms |
| S2 | Desconocido noche/perímetro | Solo luz disuasiva | Police=deny intrusivos; risk=high |
| S3 | Desconocido noche/interior | Review | Police=review; no intrusivos |
| S4 | Baja coherencia (visión vs texto) | Review | coh<0.35 → review |
| S5 | Alta coherencia noche/conocido fuerte | Allow | allow + credit>0.1 |
| S6 | Modo away + movimiento interior | Critical | risk=critical; deny si coh<strict |
| S7 | Falla de cámara | Degradación segura | coh con context; no intrusivos |
| S8 | Ruido de sensor (rebote 10s) | Anti‑replay | 1 decisión efectiva |
| S9 | Conocido con emoción negativa | Empatía + luz | notify.brief o tts.greet; no sirena |
| S10 | Intrusión confirmada | Plan crítico | allow plan crítico; evidencia camera.record si Police lo permite |
| S11 | Drift de riesgo 24h | Aprendizaje | learning_state sugiere ajuste |
| S12 | Feedback negativo usuario | Refuerzo | credit.signal ajusta eficacia |

---

## 4) Casos por escenario (detallados)

### S1 — Conocido día / perímetro
**Setup:** `zone=perimeter`, `home_mode=day`  
**Input (curl):**
```bash
curl -s -X POST http://localhost:8000/ingest/event -H "Content-Type: application/json" -d '{
  "event": {"id":"S1_001","source":"simulator","sensor":"camera.entrada_principal","zone":"perimeter","home_mode":"day"},
  "ts":1760888405.123
}'
```
**Expected:** `policy_vector` contiene `tts.greet` y/o `light.warm`; `Police=allow`; `credit>0`.  
**DoD:** NDJSON con `coherence.overall≥0.6`, latencia≤400ms; metrics.json actualizado.

---

### S2 — Desconocido noche / perímetro
**Setup:** `zone=perimeter`, `home_mode=night`  
**Expected:** `light.alert` permitido; deny para intrusivos.  
**DoD:** risk=high; Police.reason regla nocturna; credit neutro o positivo.

---

### S3 — Desconocido noche / interior
**Setup:** `zone=indoor`, `home_mode=night_armed`  
**Expected:** Police=review; sin acciones intrusivas.  
**DoD:** NDJSON con review; latencia≤400ms.

---

### S4 — Baja coherencia
**Setup:** conflicto entre modalidades (visual/texto).  
**Expected:** coh<0.35 → review.  
**DoD:** coherence baja registrada; explanation_short claro.

---

### S5 — Alta coherencia noche / conocido fuerte
**Setup:** coherencia≥0.6; known fuerte; night.  
**Expected:** allow con greeting/luz; credit>0.1.  
**DoD:** NDJSON y metrics.json reflejan éxito.

---

### S6 — Modo away + movimiento interior
**Setup:** home_mode=away; zone=indoor; presencia.  
**Expected:** risk=critical; Police según coherencia.  
**DoD:** acción coherente; registro completo con reason.

---

### S7 — Falla de cámara
**Setup:** sin visual; context.presence=true.  
**Expected:** decisiones de bajo costo; no intrusivos.  
**DoD:** NDJSON omite visual; coherencia context válida.

---

### S8 — Rebote 10s
**Setup:** 5 eventos en 10s iguales.  
**Expected:** 1 decisión válida; deduplicación sandbox.  
**DoD:** 1 NDJSON relevante; contadores correctos.

---

### S9 — Conocido con emoción negativa
**Setup:** contexto empático (día/entry).  
**Expected:** notify.brief o tts.greet suave; no sirena.  
**DoD:** credit≥0; wellbeingΔ≥0.

---

### S10 — Intrusión confirmada
**Setup:** unknown@indoor@night_armed.  
**Expected:** plan crítico permitido; camera.record opcional.  
**DoD:** NDJSON con risk=critical; evidencia si aplica.

---

### S11 — Drift de riesgo
**Setup:** aumento 5–10% eventos críticos.  
**Expected:** learning_state drift_24h actualizado.  
**DoD:** worker genera snapshot y métricas cambian.

---

### S12 — Feedback negativo usuario
**Setup:** enviar feedback −1 a /aion/feedback.  
**Expected:** credit.signal ajustado; eficacia baja siguiente ventana.  
**DoD:** learning_state top_ineffective actualizado.

---

## 5) Procedimiento general
1. Setear ENV; limpiar /results y /state.  
2. Inyectar eventos S1–S12.  
3. Revisar NDJSON, metrics.json, learning_state.json.  
4. Medir latencias y tasas.  
5. Consolidar resultados.

---

## 6) Checklist
- [ ] NDJSON tiene event, modalities, decision, metrics, police, credit, trace.  
- [ ] policy_vector canónico.  
- [ ] police.verdict y reason correctos.  
- [ ] latency_ms ≤400ms.  
- [ ] metrics.json actualizado.  
- [ ] learning_state.json actualizado.

---

## 7) Trazabilidad
| Requisito | Pruebas |
|-----------|---------|
| Invariante #0 | Todos sandbox |
| Coherencia operativa | S4, S5, S7 |
| Reglas Police | S2, S3, S6, S10 |
| Aprendizaje incremental | S11, S12 |
| Observabilidad | metrics.json |
| Privacidad | S10 (camera.record condicionado) |

---

## 8) Anexos — ejemplos y utilidades

### 8.1 Evento genérico (plantilla homologada)
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

### 8.2 Comandos útiles
- Rotación de resultados y backups.  
- Inyección por lotes (S8, S11).

---

### Notas finales
- Plan iterativo: tasas y umbrales ajustables tras primer ciclo.  
- Documentar *post‑mortems* de fallas DoD.

---

### Registro de cambios v2025-10-17
- Homologado con Master Context (contrato sensor‑centrado, sin payload).  
- Actualizados ejemplos y plantillas.  
- Estandarizadas referencias de métricas y coherencia.  
- Conservada estructura y numeración original.

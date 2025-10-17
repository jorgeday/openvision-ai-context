# AION-Genesis — Design v0.1 · Cierre Arquitectónico
**Fecha:** 2025-10-17  
**Propósito:** Documento técnico interno que consolida las decisiones arquitectónicas del diseño v0.1.  
**Relación:** Complemento directo del *AION-Genesis_Master_Context* (mantiene el Invariante #0).

---

## 1. Canon de Acciones (policy_vector)

| Acción | Propósito | Canal | Costo relativo | Ejemplo |
|--------|------------|--------|----------------|----------|
| light.warm | Luz de cortesía suave | HA | Bajo | Encender entrada para usuario conocido |
| light.alert | Luz intensa/disuasiva | HA | Medio | Parpadeo exterior ante intruso |
| notify.brief | Notificación corta | Vocalis/HA | Bajo | “Movimiento detectado en jardín” |
| notify.urgent | Alerta prioritaria | Vocalis/HA | Medio | “Posible intrusión detectada” |
| siren | Alarma sonora | HA | Alto | Intruso nocturno |
| camera.record | Captura o grabación | HA/Watcher | Medio | Registrar rostro o evidencia |
| listen.area | Activar micrófono de zona | HA/local | Alto | Escuchar sonido sospechoso |
| tts.greet | Saludo empático | Vocalis | Bajo | “Bienvenido Jorge” |

---

## 2. Taxonomía de Contexto

| Campo | Valores permitidos | Regla de canonización | Ejemplo |
|--------|-------------------|-----------------------|----------|
| zone | perimeter, entry, indoor, garage, living, bedroom | Según sensor/cámara | “camera.entrada_principal”→perimeter |
| home_mode | day, night, night_armed, away | Si alarma armada y hora nocturna→night_armed | “23:00+alarmaON”→night_armed |
| identity_bucket | unknown, known_weak, known_strong | Confianza facial (>0.8=strong,0.5–0.8=weak) | 0.93→known_strong |

---

## 3. Police v0.1 — Reglas Duras

| Condición | Veredicto | Razón |
|------------|------------|--------|
| unknown@perimeter@night(_armed) | deny (siren,camera,listen) | Falsos positivos altos |
| unknown@indoor@night(_armed) | deny todas, review | Intrusión potencial |
| coherence.overall < COH_ALERT_THR | review | Incoherencia leve |
| coherence.overall < COH_STRICT_THR + night(_armed) | deny | Incoherencia grave en horario crítico |
| known_strong@perimeter@day | allow greet/luz | Bienvenida normal |
| known_weak@perimeter@night | review | Identificación incierta |
| risk=critical + coherence>0.6 | allow plan crítico | Confirmado |
| wellbeing_delta < -0.5 | review | Acción puede ser dañina |
| Police.override_active=true | deny todo | Modo emergencia |

---

## 4. Parámetros de Aprendizaje

| Variable | Descripción | Valor |
|-----------|--------------|-------|
| LEARN_LOOKBACK_DAYS | Días de historia a procesar | 7 |
| LEARN_MAX_EVENTS | Máx. eventos por ciclo | 20000 |
| LEARN_BATCH_EVENTS | Frecuencia de ejecución | 500 |
| MIN_USAGE | Mínimo de uso de acción | 25 |
| FAIL_THR | Crédito bajo → inefectivo | -0.10 |
| DRIFT_THR | Umbral de deriva | 0.08 |
| LEARN_PERIOD_MIN | Periodicidad min | 15min |

---

## 5. Rotación y Retención

| Política | Valor |
|-----------|--------|
| Rotación diaria | `/results/YYYY-MM-DD.ndjson` |
| Compresión semanal | `.gz` >7 días |
| Retención activa | 30 días |
| Integrity hash | SHA256 opcional |

---

## 6. Matriz de Pruebas Piloto (S1–S12)

| ID | Escenario | Esperado | DoD |
|----|------------|-----------|-----|
| S1 | Conocido día/perímetro | Greet+Luz cálida | Police=allow,credit>0 |
| S2 | Desconocido noche/perímetro | Luz solo | deny, risk=high |
| S3 | Desconocido noche/interior | Review | review sin acción intrusiva |
| S4 | Baja coherencia día | Review | coh<0.35 → review |
| S5 | Alta coherencia noche/known | Allow | credit>0.1 |
| S6–S12 | Variaciones (ruido, fallos, away) | Según regla | coherencia>0.6,falsos=0 |

---

## 7. Observabilidad mínima

| Métrica | Descripción |
|----------|-------------|
| events_total | Eventos procesados |
| allow_count / deny_count / review_count | Decisiones Police |
| avg_latency_ms | Latencia promedio |
| avg_coherence_overall | Promedio global coherencia |
| avg_credit_signal | Promedio global crédito |
| conflict_rate | % incoherencias |

Archivar en `/data/aion/metrics.json` o MQTT `aion/metrics`.

---

## 8. Privacidad y Trazabilidad

| Aspecto | Regla |
|----------|-------|
| PII | Nombres→hash SHA1 |
| Video/Audio | Solo bajo camera.record |
| Logs | Sin textos reales ni voz |
| Integridad | Hash diario NDJSON |
| Exportación | Solo sandbox, token local |
| Retención | 30 días automática |

---

## 9. Interfaces Sandbox

| Endpoint | Método | Descripción |
|-----------|---------|--------------|
| /ingest/event | POST | Recibe evento HA |
| /aion/learning/state | GET | Snapshot aprendizaje |
| /aion/health | GET | Estado básico |
| /aion/feedback | POST | (futuro) evaluación usuario |

---

## 10. ENV Seed Inicial

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

### Observaciones finales
- Todas las reglas cumplen el Invariante #0 (aislamiento total respecto a CAC 2.0).  
- Este diseño queda aprobado como **baseline AION‑Genesis v0.1** para pruebas piloto y expansión futura.
  
- ### Registro de cambios v2025-10-17
- Validado y homologado con *AION-Genesis_Master_Context v2025-10*.  
- Confirmada coherencia total en canon, taxonomía, reglas Police, y ENV Seed.  
- Añadido registro de cambios (sin modificaciones estructurales).


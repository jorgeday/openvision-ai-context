# AION-Genesis — Diccionario de Datos v0.1
**Fecha:** 2025-10-17  
**Propósito:** Documento técnico de referencia que describe todos los campos, conceptos y variables empleados en el ecosistema cognitivo AION‑Genesis v0.1.  
**Relación:** Complementa el *AION‑Genesis_Master_Context v2025-10* y el documento *Cierre Arquitectónico v0.1*.

---

## 🔸 1. Eventos y Contexto

| Término | Descripción | Ejemplo |
|----------|--------------|----------|
| **event.id** | Identificador único de evento (UUID). | `9b1d3f3e‑6f7c‑4b8b‑9d3a‑221a9e0c6a33` |
| **event.source** | Origen del evento (`home_assistant` / `simulator`). | `"home_assistant"` |
| **event.sensor** | ID completo del sensor que generó el evento. | `"binary_sensor.puerta_entrada"` |
| **event.device_class** | Tipo de sensor (door, motion, contact, light, etc.). | `"door"` |
| **event.state** | Estado actual del sensor. | `"on"` |
| **event.prev_state** | Estado previo (para inferir transición). | `"off"` |
| **event.zone** | Área física/lógica del evento. | `"perimeter"`, `"living"` |
| **home_mode** | Estado global del hogar. | `"night"` |
| **timestamp (ts)** | Momento del evento en epoch segundos. | `1760888405.123` |

> **Nota:** El contrato de ingesta es **sensor‑centrado**.  
> Cualquier información sobre identidad o emoción proviene de sub‑agentes (Watcher, Vocalis) y se refleja en `modalities` dentro del **AIONResult**, no en la ingesta.

---

## 🔸 2. Modalidades (de percepción)

| Modalidad | Representa | Campos típicos | Ejemplo |
|------------|-------------|----------------|----------|
| **visual** | Cámara / rostro / emoción | `conf`, `labels`, `emotion`, `emb` | `{"labels":{"person":0.99},"emotion":{"sad":0.74}}` |
| **linguistic** | Voz o texto recibido | `text`, `conf`, `emb` | `"todo bien, solo llegué tarde"` |
| **context** | Entorno de HA (hora, luz, presencia). | `features`, `emb`, `conf` | `{"hour":"23:41","presence":true}` |
| **emb (embedding)** | Vector numérico que resume significado. | `[0.012, ‑0.045, …]` | Utilizado para medir similitud entre modalidades. |

---

## 🔸 3. Decisión (capa cognitiva)

| Campo | Significado | Ejemplo |
|--------|--------------|----------|
| **policy_vector** | Acciones con probabilidad o peso. | `{"light.warm":0.6,"notify.brief":0.3}` |
| **risk_level** | Nivel de riesgo inferido. | `"low" / "medium" / "high" / "critical"` |
| **wellbeing_delta** | Impacto esperado (–1 a 1). | `‑0.15` |
| **explanation_short** | Texto humano que explica la decisión. | `"Desconocido en perímetro de noche; luz exterior ON."` |

---

## 🔸 4. Métricas internas

| Campo | Qué mide | Comentario |
|--------|-----------|-------------|
| **coherence.vt** | Coherencia visual↔texto | Alta si lo que se ve coincide con lo que se dice. |
| **coherence.vc** | Coherencia visual↔contexto | Alta si la escena concuerda con el entorno. |
| **coherence.tc** | Coherencia texto↔contexto | Alta si la frase coincide con la situación. |
| **coherence.overall** | Promedio ponderado (0–1). | `1` = totalmente coherente. |
| **latency_ms** | Tiempo total del ciclo de decisión. | `238` ms |

---

## 🔸 5. Police y Crédito

| Campo | Significado | Ejemplo |
|--------|--------------|----------|
| **police.verdict** | Dictamen del agente ético: `allow`, `deny`, `review`. | `"deny"` |
| **police.reason** | Justificación textual. | `"Regla: unknown@perimeter@night → strict"` |
| **credit.signal** | Recompensa inmediata (–1 a 1). | `‑0.2` |
| **credit.reason** | Explicación del refuerzo. | `"Ruido innecesario evitado"` |

---

## 🔸 6. Aprendizaje Incremental (Worker)

| Campo | Significado | Ejemplo |
|--------|-------------|----------|
| **learning_state.json** | Archivo con memoria condensada diaria. | Guarda promedios y patrones. |
| **avg_coherence / avg_credit** | Promedios globales. | `0.71`, `0.08` |
| **by_context.key** | Contexto canónico (`zone@home_mode`). | `"perimeter@night"` |
| **top_effective / top_ineffective** | Acciones más y menos efectivas. | `["light.warm",0.19]` / `["siren",-0.27]` |
| **pattern_fail** | Patrones repetidos de fallo. | `["unknown@perimeter@night"]` |
| **risk.dist** | Distribución porcentual de riesgo. | `{"low":0.28,"high":0.24}` |
| **risk.drift_24h** | Cambio porcentual en 24 h. | `{"critical":+0.02}` |
| **coherence.low_conflict_rate** | % de eventos con baja coherencia. | `0.18` |
| **suggest.COH_ALERT_THR / COH_STRICT_THR** | Recomendaciones de ajuste. | `"+0.03"` |

---

## 🔸 7. Variables de Entorno (ENV)

| Variable | Propósito | Valor típico |
|-----------|------------|---------------|
| LEARN_LOOKBACK_DAYS | Ventana de historia. | 7 |
| LEARN_MAX_EVENTS | Máx. eventos por ciclo. | 20000 |
| LEARN_BATCH_EVENTS | Frecuencia de análisis. | 500 |
| MIN_USAGE / FAIL_THR | Parámetros de eficacia. | 25 / ‑0.10 |
| DRIFT_THR | Umbral de deriva. | 0.08 |
| COH_ALERT_THR / COH_STRICT_THR | Umbrales de coherencia. | 0.35 / 0.25 |
| AION_RESULTS_DIR / STATE_DIR | Rutas de archivos. | `/data/aion/results`, `/data/aion/state` |
| AION_LEARNING_TOPIC | Topic MQTT. | `"aion/learning/state"` |

---

## 🔸 8. Conceptos Clave

| Concepto | Descripción |
|-----------|-------------|
| **Coherencia** | Evalúa si las percepciones son consistentes entre sí. |
| **Credit signal** | Recompensa o penalización inmediata para aprendizaje por refuerzo. |
| **Police** | Agente ético que aprueba o bloquea acciones. |
| **Pattern fail** | Contextos repetidos donde el plan falla. |
| **Drift** | Cambio en tendencias de riesgo o crédito. |
| **Policy vector** | Conjunto de acciones decididas con ponderación. |
| **Learning worker** | Proceso que resume experiencia desde NDJSON. |
| **State file** | Memoria condensada usada en el siguiente ciclo. |

---

## 🔸 9. Ejemplo de Ciclo Completo

1. **Evento:** sensor puerta detecta movimiento a las 23:41.  
2. **Percepción:** cámara ve rostro desconocido → emoción neutral.  
3. **Análisis:** coherencia baja con contexto nocturno.  
4. **Decisión:** CAC → encender luz perimetral.  
5. **Police:** aprueba (allow).  
6. **Resultado:** acción ejecutada → registro NDJSON.  
7. **Worker:** detecta patrón `unknown@perimeter@night` con crédito bajo.  
8. **Aprendizaje:** recomienda endurecer regla Police.  
9. **Ciclo siguiente:** sistema reacciona con mayor precaución.

---

### Registro de cambios v2025-10-17
- Validado contra *AION‑Genesis_Master_Context v2025‑10*.  
- Confirmado contrato **sensor‑centrado**.  
- Ajustadas descripciones de coherencia y credit según *AIONResult v0.1*.  
- Homologada relación y encabezado.  
- Sin modificaciones estructurales.



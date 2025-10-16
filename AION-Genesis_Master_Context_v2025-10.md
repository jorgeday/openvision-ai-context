# 🧠 AION-Genesis · Master Context v2025-10  
**Proyecto Piloto – CAC 4.0 (Cognitive Action Center IA‑Nativo)**  
**Nexus Smart Solutions / OpenVision AI**  
**Fecha:** 16 de octubre de 2025  

---

## 0️⃣ Invariante #0 · Aislamiento Absoluto  
AION‑Genesis (CAC 4.0) es un **proyecto completamente independiente** del CAC 2.0.  
No comparte bases, canales ni lógica; su único propósito es **experimentar, observar y aprender** sin afectar la operación estable de OpenVision AI 2.x.  

> “Genesis observa sin tocar, razona sin interferir.”  

---

## 1️⃣ Propósito General  
Evaluar una **arquitectura cognitiva jerárquica IA‑nativa**, en la que un **meta‑agente central** coordina sub‑agentes especializados (visión, lenguaje, emoción, contexto, ética, ejecución).  
El piloto busca demostrar razonamiento multimodal, aprendizaje continuo y coherencia ética antes de conectarse a sensores reales.  

---

## 2️⃣ Arquitectura Conceptual  

```
[Home Assistant → eventos mínimos]
           ↓
[CAC 4.0 · Meta‑Agente]
           ↓
[Sub‑Agentes Cognitivos]
           ↓
[Registro Cognitivo + Evaluación]
```

- **Entrada:** eventos mínimos desde Home Assistant (uno por sensor).  
- **Procesamiento:** coordinación de sub‑agentes por el meta‑agente.  
- **Salida:** registro cognitivo, decisiones, métricas y trazas explicables.  

---

## 3️⃣ Ontología Cognitiva  
**Objetos básicos**: entidad · evento · contexto · emoción · intención · acción · política.  
Cada objeto se representa como **vector (embedding 1024‑D)** dentro del espacio cognitivo común.  

---

## 4️⃣ Contrato Mental del Meta‑Agente  
**Entradas:** embeddings sensoriales, contextuales y emocionales + guardrails.  
**Salidas:** `policy_vector`, `risk_level`, `wellbeing_delta`, `explanation_short`, `credit_signal`.  

**Cálculo de crédito:**  
`trust_next = trust_current + α*(Δ_bienestar - Δ_riesgo) - β*Police_override`  

**Leyes internas (L1–L7)**  
1. Protección de la vida humana.  
2. No causar perturbación física ni psicológica.  
3. Mantener privacidad y minimizar exposición.  
4. Requiere doble señal para acciones invasivas.  
5. Priorizar bienestar colectivo.  
6. Registrar trazas explicables sin PII.  
7. Obedecer órdenes humanas solo si no contradicen L1–L6.  

---

## 5️⃣ Sub‑Agentes Cognitivos  

| Sub‑Agente | Rol | Disparadores típicos |
|-------------|-----|----------------------|
| **Visual** | Percibe entorno, rostros, emociones. | Eventos `camera` o `motion`. |
| **Lingüístico / Vocalis** | Analiza texto o voz; extrae intención. | Entrada de voz / transcripción. |
| **Emocional** | Integra estados afectivos globales. | Cada ciclo cognitivo. |
| **Contextual** | Evalúa tiempo, zona, rutina, anomalías. | Cualquier sensor. |
| **Ético (Police)** | Aplica leyes L1–L7 antes de actuar. | Previo a ejecución. |
| **Motor** | Ejecuta planes físicos o virtuales aprobados. | Tras autorización ética. |

---

## 6️⃣ Flujo Cognitivo  
```
Evento HA → Enriquecimiento contextual → Invocación de sub‑agentes
→ Fusión vectorial → Planificación → Police → Acción/Registro
```
Latencia objetivo ≤ 400 ms.  

---

## 7️⃣ Protocolo HA → CAC 4.0 (v0.1‑min)

```json
{
  "schema_version": "0.1",
  "event_id": "<uuid>",
  "ts": <epoch_ms>,
  "source": "home_assistant",
  "sensor_domain": "<sensor|binary_sensor|lock|camera|...>",
  "sensor_id": "<entity_id>",
  "device_class": "<door|motion|temperature|lock|...>",
  "state": "<value>",
  "prev_state": "<value>",
  "location": { "zone": "<perimeter|interior>", "room": "<nombre_zona>" }
}
```
- Cada evento representa **un único sensor físico**.  
- HA solo informa estado; el CAC 4.0 **interpreta y enriquece**.  

---

## 8️⃣ Protocolo Interno de Emulación (v0.1)  
Usado en el sandbox para cerrar el ciclo de aprendizaje.  
En operación real, este feedback será reemplazado por datos reales del entorno.  

```json
{
  "header": { "agent_id": "...", "agent_type": "...", "timestamp": 1759999 },
  "payload": { "embeddings": { "dim": 1024 }, "confidence": 0.9 },
  "feedback": { "approved": true, "credit": { "delta_bienestar": 0.1 } }
}
```

---

## 9️⃣ Matriz de Escenarios de Prueba Cognitiva (v0.1)

| ID | Contexto / Objetivo | Evento HA mínimo | Sub‑agentes | Métricas |
|----|----------------------|------------------|--------------|-----------|
| S1 | Puerta perímetro noche (básico) | door on | Contextual, Visual, Ético, Motor | Lat≤400 ms, FP/FN, Expl≥4 |
| S2 | Movimiento perímetro sin puerta | motion on | Contextual, Visual | FP reducción |
| S3 | Rebote puerta <1 s | door on/off rápido | Contextual, Ético | dedup efectivo |
| S4 | Movimiento pasillo día | motion on interior | Contextual, Emocional | acción soft |
| S5 | Inactividad living | motion off prolongado | Contextual, Emocional, Ético | sugerencia pertinente |
| S6 | Temperatura alta | temp 29.5 °C | Contextual, Emocional | Δ bienestar + |
| S7 | Cerradura desbloqueada | lock unlocked | Contextual, Visual, Ético | clasificación correcta |
| S8 | Movimiento interior noche | motion on @night | Contextual, Visual, Ético | lat + FP |
| S9 | Cámara detecta persona | camera person | Visual, Contextual, Ético, Motor | plan proporcional |
| S10 | Cámara idle + puerta on | door on + camera idle | Visual, Contextual | coherencia |
| S11 | Voz “estoy agotado” | sensor.intent_text | Lingüístico, Emocional, Contextual | Δ bienestar + |
| S12 | Acción invasiva noche | cualquier noche + sirena | Ético (Police) | bloqueo correcto |

**Ciclo de simulación**  
1. Inyectar evento mínimo HA→CAC4.  
2. Observar sub‑agentes invocados.  
3. Registrar latencia, plan, riesgo, Δ bienestar, Police, explicación.  
4. Comparar con criterio esperado.  
5. Ajustar pesos y repetir.  

---

## 🔟 Métricas clave de evaluación  
- `latency_ms` (p95)  
- `risk_false_pos / risk_false_neg`  
- `wellbeing_delta (avg)`  
- `Police_override_rate (%)`  
- `explanation_score (1–5)`  
- `coherence_score (0–1)`  

---

## 🔢 Estado actual del proyecto
| Elemento | Estado |
|-----------|---------|
| Invariante #0 | ✅ Activa |
| Contrato Mental | ✅ Definido |
| Sub‑Agentes | ✅ Definidos |
| Protocolo HA→CAC v0.1‑min | ✅ Definido |
| Matriz de Escenarios v0.1 | ✅ Definida |
| Próximo paso | Diseñar formato de registro de resultados y métrica de coherencia |

---

## 📘 Uso del documento  
Este contexto maestro define la arquitectura y las reglas del piloto AION‑Genesis.  
Debe cargarse al inicio de cualquier sesión relacionada con CAC 4.0 para mantener la coherencia cognitiva del desarrollo.  
El documento no reemplaza OpenVision Master Context v2.x; lo complementa como rama experimental del sistema.  

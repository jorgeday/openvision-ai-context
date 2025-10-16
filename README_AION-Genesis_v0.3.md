# 🧠 AION-Genesis · Cognitive Action Center 4.0  
**Proyecto piloto aislado – Nexus Smart Solutions / OpenVision AI**  
**Versión:** v0.3 (16 Oct 2025) **Estado:** Sandbox Cognitivo  

---

## 🌐 Descripción general
**AION-Genesis** es el laboratorio cognitivo de OpenVision AI.  
Implementa el **CAC 4.0 (Cognitive Action Center IA-nativo)** como sistema totalmente independiente del CAC 2.0.  
Su objetivo es **evaluar un modelo jerárquico multimodal** donde un **meta-agente central** coordina sub-agentes especializados (visión, lenguaje, emoción, contexto, ética, ejecución).

> “Genesis observa sin tocar, razona sin interferir.” — Invariante #0

---

## ⚙️ Propósito del repositorio
1. **Experimentar** la mente cognitiva 4.0 sin conectar hardware real.  
2. **Emular** sub-agentes para probar coordinación, coherencia y ética.  
3. **Validar** la arquitectura jerárquica IA-nativa antes de integrarla al entorno operativo.  

---

## 🧩 Estructura de carpetas sugerida
```
/docs
   └── AION-Genesis_Documento_Maestro_v0.3.md   ← referencia principal
/tests
   └── scenarios/                               ← escenarios de prueba cognitiva
/logs
   └── cognition/                               ← resultados y métricas
/src
   ├── adapters/                                ← gateway HA→CAC (simulación)
   ├── meta_agent/                              ← núcleo del CAC4 (razonamiento)
   └── sub_agents/                              ← visual, contextual, ético, etc. (emulados)
README.md
```

---

## 🔒 Invariante #0 · Aislamiento
AION-Genesis **no comparte** bases, canales ni servicios con CAC 2.0.  
Funciona en su propio stack (contenedor o entorno local).  
Toda comunicación externa es **solo lectura** proveniente de Home Assistant.

---

## 🔗 Integración mínima HA → CAC 4.0
Cada evento representa **un sensor**:
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

---

## 🧠 Componentes cognitivos
| Módulo | Función principal |
|--------|-------------------|
| **Meta-Agente** | Orquestador jerárquico; interpreta eventos y decide planes. |
| **Visual** | Percepción de personas, objetos y emociones faciales. |
| **Lingüístico / Vocalis** | Comprensión de voz y texto; generación de intención. |
| **Emocional** | Fusión afectiva global; clima emocional del hogar. |
| **Contextual** | Rutinas, horarios, presencia y anomalías. |
| **Ético (Police)** | Aplicación de las leyes internas L1–L7. |
| **Motor** | Ejecución física o virtual de planes aprobados. |

---

## 🧪 Matriz de Escenarios de Prueba Cognitiva
12 escenarios definidos (S1–S12) cubren seguridad perimetral, confort, ética y bienestar.  
Cada escenario inyecta un evento HA→CAC4.0 y evalúa:
- latencia (≤ 400 ms)  
- coherencia de plan  
- Δ bienestar  
- Police override rate  
- calidad de explicación  

Ver detalles en el **Documento Maestro v0.3**.

---

## 📊 Próximos pasos
- Diseñar **formato de registro de resultados** y métrica de coherencia cognitiva.  
- Implementar **simulador de escenarios** que consuma los JSON mínimos HA→CAC4.0.  
- Analizar patrones emergentes antes de conectar sub-agentes reales.  

---

## 📚 Referencia principal
➡️ `/docs/AION-Genesis_Documento_Maestro_v0.3.md`  
Contiene todos los invariantes, contratos, protocolos y matriz de pruebas.  
Debe cargarse en cada sesión futura para mantener coherencia de desarrollo.

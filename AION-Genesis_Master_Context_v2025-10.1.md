# 🧠 AION-Genesis — Master Context (v2025-10.1)
**Estado:** Piloto cognitivo aislado (AION-Genesis 4.0) — **Invariante #0** vigente  
**Propósito:** Contexto maestro para diseño, pruebas y evolución del meta-agente cognitivo **AION-Genesis**, separado 100 % de CAC 2.0 para observar, aprender y mejorar sin afectar el entorno operativo.  
**Fecha de actualización:** Octubre 2025  
**Autor:** Nexus Smart Solutions / OpenVision AI

---

## 0) Invariante #0 (AION-Genesis Invariant)
> **Aislamiento total de CAC 2.0.**  
> Sin compartir bases, canales, servicios ni lógica. Observa sin tocar. Todo despliegue de AION-Genesis es *sandbox* y no interfiere con el stack 2.x.

---

## 1) Objetivo y alcance (v0.1)
- Meta-agente IA-nativo que coordina sub-agentes (visual, lingüístico, contextual, ético/Police, aprendizaje).  
- Ciclo cognitivo completo: evento → percepción → coherencia → decisión → Police → ejecución → registro → aprendizaje → adaptación.  
- Latencia objetivo: p50 ≤ 300 ms · p95 ≤ 400 ms.  
- Reglas duras (Police) siempre sobre cualquier heurística o aprendizaje.

---

## 2) Arquitectura mínima (v0.1)
- Entradas: eventos mínimos desde Home Assistant (HA) o simulador.  
- Motor cognitivo: embeddings multimodales (VT/VC/TC), riesgo + bienestar → `policy_vector`.  
- Police: `allow|deny|review` según reglas duras.  
- Ejecución: modo sandbox, acciones reversibles.  
- Memoria: registros NDJSON (`AIONResult v0.1`) y `learning_state.json`.  
- Aprendizaje incremental: ajuste continuo sin hardcode.

---

## 3) Protocolo de evento mínimo (ingesta) — (Contrato oficial)
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
- Contrato **sensor-centrado**.  
- Identidad, emoción o texto provienen de sub-agentes y se reflejan solo en `modalities` del `AIONResult`, no en la ingesta.  
- `zone` ∈ {perimeter, entry, indoor, garage, living, bedroom}.  
- `home_mode` ∈ {day, night, night_armed, away}.

---

## 4) Formato de registro — **AIONResult v0.1**
*(sin cambios, se mantiene conforme a v2025-10 original)*

---

## 5 – 11) Coherencia, Aprendizaje, Police, Acciones, Contexto, APIs, Observabilidad, Privacidad  
*(idénticos al documento v2025-10)*

---

## 🧩 Anexo A — Catálogo base de eventos HA (guía referencial)
| Evento HA | `event.sensor` (ejemplo HA) | `device_class` | `state` / `prev_state` | `zone` | `home_mode` |
|---|---|---|---|---|---|
| Movimiento detectado | `binary_sensor.pasillo_norte_motion` | `motion` | `on/off` | `indoor` | `day|night|away|night_armed` |
| Ocupación on/off | `binary_sensor.living_occupancy` | `occupancy` | `on/off` | `living` | idem |
| Puerta/ventana abre/cierra | `binary_sensor.perimetro_puerta_entrada`, `binary_sensor.perimetro_ventana_loggia_norte`, `binary_sensor.perimetro_ventana_oficina`, etc. | `door` / `opening` | `on/off` | `perimeter` | idem |
| Timbre presionado | `binary_sensor.video_doorbell` | `door` | `on/off` | `perimeter` | `day|night` |
| Línea perimetral | `camera.frontyard_line` | `motion` | `on/off` | `perimeter` | `night|away|night_armed` |
| Persona detectada (vis) | `camera.frontyard_person` | `motion` | `on/off` | `perimeter` | idem |
| Tamper cámara | `camera.frontyard` | `tamper` | `on/off` | `perimeter` | `night|away|night_armed` |
| Humo detectado | `sensor.humo_cocina` | `smoke` | `on/off` | `indoor` | todos |
| Temperatura | `sensor.temp_sala` | `temperature` | `xx.x` | `living` | todos |
| Luz on/off | `light.sala` | `light` | `on/off` | `living` | todos |
| Media play/pause | `media_player.living_tv` | `media` | `play/pause/stop` | `living` | todos |
| Persona llega/se va | `person.jorge` / `device_tracker.life360_jorge` | `presence` | `home/not_home` | `global` | — |
| Modo hogar cambia | `input_select.home_mode` | `mode` | `day|night|away|night_armed` | `global` | — |

> *La ubicación completa (ej. `ventana_loggia_norte`) siempre vive en `event.sensor` y no se pierde.*

---

## 🧮 Anexo B — Notas operativas para CAC (internas)
- El CAC **recibe y conserva** el objeto `event` tal cual contrato §3.  
- Puede derivar internamente tipos o episodios → no contratual.  
- Sub-agentes añaden modalidades (visual, linguistic, context).  
- Identidad/emoción se registran solo en `AIONResult.modalities`.

---

## 🔒 12.b) Invariante técnica — Ingesta *Lossless* del CAC
- El CAC **no muta** `event.*`.  
- Todos los campos de entrada se copian 1:1 a `AIONResult.event`.  
- QA: `hash(input.event)==hash(AIONResult.event)` → `lossless_ingest_rate=100 %`.  
- Si desciende, `Police.review` bloquea despliegue.  
- La ubicación exacta (ej. `perimetro_ventana_loggia_norte`) se preserva y se usa para explicaciones y aprendizaje.

---

### ✅ Resumen de estado actual
| Sección | Estado |
|----------|---------|
| §3 Contrato de ingesta | Original, sin cambios |
| Catálogo de eventos HA | Agregado (guía no contractual) |
| Reglas de nomenclatura de sensores | Aplicadas (área_tipo) |
| Política de autonomía sandbox | Definida (Nivel 1: simulación limitada) |
| Principio Lossless | Aprobado e incorporado |
| Flujo CAC → AION ①–⑨ | Documentado para operación interna |

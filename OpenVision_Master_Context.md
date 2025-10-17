# OpenVision AI — Master Context (v2.x · Stable Branch)

**Fecha de última sincronización:** 2025-10-17  
**Autor:** Arquitectura OVAI / Nexus Smart Solutions  
**Estado:** Stable · Línea 2.x congelada para investigación AION‑Genesis  

---

## 0. Propósito
Documento maestro que consolida la arquitectura, políticas y alcance operativo de **OpenVision AI 2.x**, sistema de orquestación cognitiva para entornos domésticos y profesionales.

Este documento define las bases de la línea estable y marca la separación formal respecto a **AION‑Genesis**, bajo el **Invariante #0**.

> **Invariante #0:** AION‑Genesis evoluciona como entorno cognitivo independiente, sin heredar componentes, runtime o canales de OpenVision AI 2.x.

---

## 1. Arquitectura Modular

| Módulo | Función principal | Estado actual | Observaciones |
|---------|------------------|----------------|----------------|
| **Watcher** | Captura y análisis visual (detección, reconocimiento, emoción) | Operativo | YOLOv8 + InsightFace integrado |
| **CAC (Cognitive Action Center)** | Núcleo de decisión, orquestación y política de riesgo | **v2.0 estable** | Base cognitiva convencional (no IA pura) |
| **Vocalis** | Canal LLM/voz para interacción con el usuario (Alexa/HA) | Integrado | Gateway LLM consolidado |
| **Home‑Adapter** | Integración con Home Assistant y ecosistemas domóticos | Estable | Drivers sincrónicos y seguros |
| **Notifier** | Difusión y escalamiento de eventos/alertas | Estable | Canal de broadcast inteligente |

> Próxima generación en investigación: **CAC 3.0 (IA pura)** bajo el marco **AION‑Genesis**.  
> No integrado en esta línea estable.

---

## 2. Línea de Tiempo y Estado

| Fase | Fecha | Estado | Notas |
|------|--------|---------|-------|
| CAC v1.x → v2.0 | 2025‑03 → 2025‑09 | ✅ Finalizado | Incorporación de arquitectura modular y risk scoring |
| Watcher + Vocalis | 2025‑05 | ✅ Integrado | Full stack de visión y voz operativo |
| Línea AION‑Genesis | 2025‑10 | 🧠 En curso | Proyecto independiente de investigación IA pura |

---

## 3. Principios no negociables

- **Zero‑Trust:** autenticación mTLS, tokens firmados y allowlists.  
- **Idempotencia:** `event_id` único, control anti‑replay.  
- **Data minimization:** retención corta, opt‑in para aprendizaje.  
- **Observabilidad:** trazas, métricas y auditoría con hash de evidencias.  
- **Fail‑soft:** degradación controlada ante fallas críticas.  
- **Compatibilidad Home Assistant:** eventos nativos MQTT/REST sin latencia.  

---

## 4. Política de versionado

| Línea | Estado | Descripción |
|--------|---------|-------------|
| **2.x (OpenVision AI)** | 🟢 Activa | Versión estable operativa actual |
| **3.x (AION‑Genesis)** | 🧩 Experimental | Núcleo IA autónomo en investigación |

---

## 5. Créditos

**Arquitectura:** Nexus Smart Solutions / OpenVision AI Team  
**Colaboradores:** Jorge Day, equipo de desarrollo IA‑Edge, colaboradores Home Assistant  
**Repositorio:** [github.com/jorgeday/openvision‑ai‑context](https://github.com/jorgeday/openvision-ai-context)

---

**© 2025 Nexus Smart Solutions — Todos los derechos reservados.**  
Documento interno, uso técnico y de investigación autorizado.

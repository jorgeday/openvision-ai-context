# 🧠 OpenVision AI — Contexto Maestro
**Marca:** Nexus Smart Solutions  
**Proyecto:** OpenVision AI — Plataforma Cognitiva de Domótica Emocional  
**Versión documento:** 1.0  
**Última actualización:** 15-oct-2025  

---

## ⚙️ Estado General
- **Baseline congelado:** CAC 2.0 (26-sep-2025 · MVP Pro 85%)  
- **Rama activa:** AION Pilot — Meta-agente IA-nativo  
- **Objetivo actual:** crear y entrenar desde cero un agente IA unificado (visión, voz, contexto) con policy network y verificador IA (πθ + Rψ).  
- **Modo operativo:** laboratorio (1 solo proceso, sin Home Assistant ni microservicios).  

---

## 🧩 Principios Arquitectónicos Duro-core
1. **Sin hardcode ni archivos JSON externos.**  
   Toda configuración debe ser dinámica o aprendida.  
2. **Aprendizaje IA-nativo.**  
   Las políticas y límites se aprenden de feedback humano, no de reglas.  
3. **Escalabilidad progresiva.**  
   Piloto AION corre en un proceso local, pero debe poder escalar a clúster.  
4. **Privacidad total.**  
   Todo procesamiento local; nada se envía a la nube sin consentimiento.  
5. **Human-in-the-loop.**  
   El sistema aprende de preferencias, recompensas y correcciones humanas.  

---

## 🧠 Módulos activos
| Módulo | Descripción | Estado |
|---------|--------------|--------|
| πθ (Policy Core) | Decide rol/plan/acción | Entrenando v0.1 |
| Rψ (Reward/Verifier) | Evalúa decisiones y genera rationale | Entrenando v0.1 |
| Perception.Vision | Señales sintéticas (identidad/emoción) | Activo (mock) |
| Perception.Voice | Intención y tono textual | Activo (mock) |
| Context | Zona, modo, hora | Activo |
| Memory | Buffer RAM de episodios n=100 | Activo |
| Feedback | Reward humano (-1,0,+1) | En pruebas |

---

## 🧾 Hitos
| Fecha | Hito | Estado |
|--------|-------|--------|
| 26-sep-2025 | CAC 2.0 congelado (MVP Pro 85%) | ✅ |
| 15-oct-2025 | Inicio AION Pilot (meta-agente IA-nativo) | ✅ |
| — | Entrenamiento inicial πθ y Rψ | ⏳ |
| — | Integración percepción real | ⏳ |
| — | Validación de usabilidad y comportamiento emergente | ⏳ |

---

## 🧭 Comandos de Anclaje
| Comando | Acción |
|----------|--------|
| **“Cargar estado OVAI-CAC-2.0”** | Restaura baseline congelado del sistema modular. |
| **“Cargar estado AION-Pilot-Fase1”** | Activa entorno meta-agente IA-nativo. |
| **“Continuar AION Pilot con πθ y Rψ”** | Reanuda entrenamiento del policy + verifier. |

---

## 🔒 Reglas Operativas del Piloto
- **Todo corre localmente.**  
- **No hay persistencia automática.** Los modelos se guardan manualmente (`models/policy_theta_vX.pt`).  
- **Feedback manual obligatorio:** cada decisión se valida con `reward ∈ {-1,0,+1}`.  
- **Dry-run por defecto:** sin ejecución física hasta validar seguridad.  

---

## 🧰 Estructura sugerida

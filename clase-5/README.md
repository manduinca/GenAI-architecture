# Clase 5 — Despliegue y Producción

> Clase 5 (de 5) del curso *Arquitectura de Aplicaciones con IA Generativa* — EscuelaIT.
> Duración: 90 min. Nivel: desarrolladores intermedio-avanzados.

---

## Objetivo

Al terminar la sesión, el asistente debe ser capaz de:

1. Justificar las decisiones de **infraestructura** detrás de una app LLM real (cómputo, red, dominio, TLS, despliegue, secretos) y descartar alternativas con criterio.
2. Leer y modificar el stack `Dockerfile + docker-compose + nginx` de una aplicación LLM con streaming SSE, sin romper el flujo del usuario.
3. Provisionar y desplegar end-to-end en AWS con scripts reproducibles (`setup → deploy → teardown`), entendiendo cada recurso AWS elegido y por qué.
4. Distinguir **observabilidad** mínima viable (latencia + health) de la observabilidad LLM real (tokens, coste, traces) y reconocer qué falta cuando solo tienes la primera.
5. Diseñar un **golden set** de evaluación, medir retrieval (precision@k, MRR) y calidad de respuesta (LLM-as-judge), y usarlo como gate antes de cambiar prompt o modelo en producción.

---

## Estructura (90 min)

| # | Bloque | Min | Contenido | Dónde |
|---|---|---|---|---|
| 0 | **Apertura + recap clases 1-4** | 5 | Dónde encaja "producción" en el stack ya construido. Hilo narrativo: *de demo que corre a sistema que se opera*. | — |
| 1 | **Walkthrough de infra real** | 35 | Tour comentado del repo `yana-killa-demo`: AWS (EC2, gp3, SG, Cloudflare), Docker (api, web, nginx), nginx para SSE, deploy via rsync, secretos, swap, AMI. **El instructor lleva las decisiones, no el código.** | `contenido/infra-walkthrough.md` + repo del demo |
| 2 | **Lo que la infra NO cubre** | 8 | Mapa de brechas: observabilidad LLM (tokens/coste/trace), guardrails, evaluación. Por qué `latency_ms` no es observabilidad. | — |
| 3 | **Evaluación: conceptos** | 12 | Qué es un golden set. Offline vs online. Tres niveles: retrieval / generación / end-to-end. Métricas (precision@k, MRR, hit rate). LLM-as-judge con rubric. | `notebook/eval-golden-set.ipynb` §1-2 |
| 4 | **Demo — Golden set en vivo** | 20 | Mini corpus + retriever didáctico, golden set de 8 preguntas, métricas de retrieval, LLM-as-judge con DeepSeek, comparación entre dos modelos. | `notebook/eval-golden-set.ipynb` §3-7 |
| 5 | **Cómo llevar la eval a CI/CD** | 5 | Gate de regresión: cambio de prompt/modelo → eval contra golden set → bloquear o aprobar. Coste y cadencia razonables. | — |
| 6 | **Cierre del curso** | 5 | 5 take-aways acumulados de las 5 clases. Qué leer y practicar después. Q&A. | — |

---

## Setup

El entorno es **compartido entre todas las clases del curso** (venv, `.env` y `requirements.txt` en el raíz).

```bash
# Desde la raíz del curso — una sola vez
cd /path/to/GenAI-architecture
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # completa DEEPSEEK_API_KEY

# Para esta clase
jupyter lab clase-5/notebook/eval-golden-set.ipynb
```

Para el bloque 1 (walkthrough de infra) **no se ejecuta nada en vivo**. Es lectura comentada del repo `yana-killa-demo` con la guía conceptual abierta al lado. No hace falta tener AWS configurado en clase.

---

## Tres mensajes que deben llevarse

1. **Producción es decisiones, no plantillas.** Cada elección de infra (t3.medium, gp3, Cloudflare proxied, deploy por rsync) responde a un tradeoff explícito. Si no sabes el porqué, no sabes cuándo cambiar.
2. **Observabilidad de una app LLM no es solo `latency_ms` y `/health`.** Tokens, coste, trace por request y eval continua son la diferencia entre saber y suponer. Lo que no mides no mejora.
3. **Sin golden set, "mejorar el prompt" es mover en la oscuridad.** La eval es el contrato: define qué significa "bien" antes de tocar nada.

---

## Conexión con otras clases

- **Viene de:** clase 4 (la app `yana-killa-demo` ya construida — auth, rate limit, modos de fallo Plan A/B/C, streaming SSE, citas verificables).
- **Cierra el curso:** la clase 5 contesta el "y ahora cómo se opera esto" que la clase 4 dejó abierto, y entrega la herramienta (golden set) que permite iterar el resto del sistema sin retroceder.

---

## Caso de estudio

El bloque 1 se apoya íntegramente en la infraestructura real del proyecto `yana-killa-demo` ya presentado en clase 4. La guía conceptual `contenido/infra-walkthrough.md` explica primero **el porqué** de cada decisión; recién después se proyecta el código. El objetivo es que cuando se abra `infra/setup-prod.sh` los asistentes ya sepan qué van a ver y por qué.

El bloque 4 (notebook) usa un mini corpus didáctico inspirado en el dominio del demo (relaves mineros, hidrogeología) pero autocontenido: no requiere tener el demo corriendo y reutiliza la stack de embeddings de la clase 3 (MiniLM + SQLite + DeepSeek).

---

## Tres ideas clave del curso completo (para el cierre)

1. **El LLM es infraestructura, no feature.** La diferenciación vive alrededor (clase 1).
2. **Prompt + estructura + retrieval son código versionable.** No arte oculto en strings (clases 2-3).
3. **Lo que separa demo de producto es el borde y la medida.** Streaming, citas, modos de fallo, observabilidad y eval (clases 4-5).

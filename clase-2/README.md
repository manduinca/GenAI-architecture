# Clase 2 — Prompt Engineering como sistema

> Clase 2 (de 5) del curso *Arquitectura de Aplicaciones con IA Generativa* — EscuelaIT.
> Duración: 90 min. Nivel: desarrolladores intermedio-avanzados.

---

## Objetivo

Al terminar la sesión, el asistente debe ser capaz de:

1. Distinguir zero-shot, few-shot, chain-of-thought y self-consistency, y saber cuándo usar cada una.
2. Diseñar un system prompt con estructura (rol, contexto, restricciones, formato) y defender por qué cada bloque reduce errores.
3. Forzar output estructurado con JSON mode, Pydantic + Instructor, y manejar validación + truncamiento.
4. Orquestar un pipeline multi-etapa donde cada llamada LLM es una función testeable y versionable.

---

## Estructura (90 min)

| # | Bloque | Min | Contenido | Dónde |
|---|---|---|---|---|
| 0 | **Cierre clase 1 + demo gancho** | 15 | Diapos 20–35 del pptx de clase 1 (modo rápido). Cierra bloques 3–5 y presenta la agenda de hoy con slide 35. | `clase-1/Presentation1.pptx` |
| 1 | **Setup + anatomía rápida** | 5 | `.env`, cliente DeepSeek, test de conexión, roles y temperature. | `notebook/prompting.ipynb` §0–1 |
| 2 | **Técnicas de prompting** | 20 | Zero-shot, few-shot, CoT, self-consistency con ejemplos electorales. | Notebook §2 |
| 3 | **System prompts como arquitectura** | 10 | Rol · contexto · restricciones · formato. Antes/después. | Notebook §3 |
| 4 | **Output estructurado** | 15 | JSON mode, Pydantic + Instructor, validación con retry, truncamiento. | Notebook §4 |
| 5 | **Pipeline multi-etapa** | 15 | Extracción → clasificación → informe. Cada etapa es un LLM call tipado. | Notebook §5 |
| 6 | **Puente al código Go de `peru-elecciones`** | 8 | Mapear los patrones del notebook al código real: cliente agnóstico, template de prompt, inyección de contexto SQLite. | `/Users/manduinca/Projects/deepskill/peru-elecciones` |
| 7 | **Cierre + preview clase 3** | 2 | 3 take-aways + transición a RAG y orquestación. | — |

---

## Setup

El entorno es **compartido entre todas las clases del curso** (venv, `.env` y `requirements.txt` viven en el raíz). Si ya lo preparaste para otra clase, salta a `jupyter lab`.

```bash
# Desde la raíz del curso — una sola vez
cd /path/to/GenAI-architecture
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # y completa DEEPSEEK_API_KEY

# Para esta clase
jupyter lab clase-2/notebook/prompting.ipynb
```

---

## Tres mensajes que deben llevarse

1. **Los prompts son código.** Versionados, testeables, iterados con datos — no arte oculto en un string.
2. **El system prompt bien estructurado reduce más alucinaciones que `temperature=0`.** Rol, contexto, restricciones y formato son contratos explícitos.
3. **Structured output + validación es la frontera entre demo y producto.** JSON mode abre la puerta; Pydantic + Instructor la cierra con retry.

---

## Conexión con otras clases

- **Viene de:** clase 1 (anatomía API, roles, OpenAI-compatible, DeepSeek como proveedor).
- **Lleva a:** clase 3 (cuando el pipeline deja de ser lineal: ramas, memoria, RAG, orquestación tipo grafo).

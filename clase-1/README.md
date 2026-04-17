# Clase 1 — Arquitectura IA Generativa y el Ecosistema LLM

> Clase 1 (de 5) del curso *Arquitectura de Aplicaciones con IA Generativa* — EscuelaIT, abril 2026.
> Duración: 90 min. Gratuita. Nivel: desarrolladores intermedio-avanzados.
> URL del curso: https://escuela.it/cursos/curso-ia-generativa

---

## Objetivo de la clase

Al terminar los 90 min, el asistente debe ser capaz de:

1. Explicar qué es un LLM / modelo fundacional y cómo se diferencia del ML clásico.
2. Entender tokenización, ventanas de contexto y cómo impactan en coste y calidad.
3. Navegar el ecosistema actual de proveedores (frontier US, frontier chino, open-weights, inferencia especializada).
4. Reconocer cuándo usar un framework (LangChain, LlamaIndex, Vercel AI SDK, MCP) y cuándo NO.
5. Hacer una llamada a chat completions en Python, entendiendo roles, streaming, tool use y structured output.
6. Cambiar de proveedor cambiando únicamente `base_url` (patrón del estándar OpenAI-compatible).

---

## Estructura (90 min)

| # | Bloque | Min | Contenido | Archivo |
|---|---|---|---|---|
| 0 | **Apertura + demo gancho** | 5 | Demo en vivo del asistente de `peru-elecciones` → "esto es lo que vamos a saber construir". | — |
| 1 | **Fundamentos LLM** | 15 | Modelo fundacional, Transformer, pre/post-training, capacidades emergentes y límites. Panorama de modelos frontier (Claude 4.x, GPT-5, Gemini 2.x/3, DeepSeek, Llama 4, Qwen, Mistral). Chat vs razonador. | [01-fundamentos.md](contenido/01-fundamentos.md) |
| 2 | **Tokens y contexto** | 15 | BPE, tiktoken, SentencePiece. "Language tax". Ventanas 1M+ y límites efectivos (Lost in the Middle, RULER). Prompt caching. | [02-tokens-contexto.md](contenido/02-tokens-contexto.md) |
| 3 | **Ecosistema de proveedores** | 15 | Mapa 6 capas: frontier US/CN, open-weights, inferencia (Groq/Cerebras), hyperscalers, self-host. Tabla de pricing por tier. Compliance (GDPR, EU AI Act, HIPAA). Criterios de elección. | [03-proveedores.md](contenido/03-proveedores.md) |
| 4 | **APIs de chat completions** | 20 | Anatomía de `/v1/chat/completions`. El estándar OpenAI-compatible y sus caveats. Anthropic Messages API. Streaming SSE. Tool use. Structured output. Demo: misma llamada a 2 proveedores cambiando solo `base_url`. | [05-apis.md](contenido/05-apis.md) |
| 5 | **Frameworks: cuándo sí, cuándo no** | 10 | LangChain/LangGraph, LlamaIndex, Semantic Kernel, Haystack, Pydantic AI, Vercel AI SDK, DSPy. MCP como estándar emergente. Postura del curso: empezar con SDK directo. | [04-frameworks.md](contenido/04-frameworks.md) |
| 6 | **Cierre + preview clase 2** | 10 | 3 take-aways + Q&A + transición a Prompt Engineering. | — |

**Hilo narrativo:** *"De ChatGPT la caja mágica al endpoint HTTP que controlamos"* — cada bloque desmitifica una capa (el modelo → el token → el proveedor → la API → el framework).

---

## Tres mensajes que deben llevarse

1. **El LLM es una capa de infraestructura, no una feature.** La diferenciación vive *alrededor* (prompts, datos, tooling, eval, UX). Justifica el curso entero.
2. **No hay "un mejor modelo".** Hay uno mejor para *tu* tarea, precio y latencia. Habilidad clave: saber elegir y saber cambiar.
3. **Las limitaciones del modelo son la razón del resto del curso.** RAG, tool use, memoria, orquestación, evaluación son respuestas arquitectónicas a fallos conocidos.

---

## Demos en vivo planificadas

1. **Demo gancho (apertura).** Asistente IA de `peru-elecciones` respondiendo preguntas reales sobre conteo ONPE. Subraya: "al final de este curso construyes esto".
2. **Tokenización comparada** (Bloque 2, 5-8 min). `tiktokenizer.vercel.app` con frases ES / EN / ZH / código → muestra el "language tax" y la diferencia cl100k_base vs o200k_base.
3. **Llamada dual OpenAI-compatible** (Bloque 4, 10 min). Mismo código Python, dos proveedores (OpenAI y DeepSeek o Groq), cambiando únicamente `base_url` y `api_key`.
4. **Leaderboards vivos** (Bloque 1 o 3, 3 min). Abrir `lmarena.ai` y `artificialanalysis.ai` para mostrar estado del arte "en vivo".

---

## Contenido detallado (dossiers)

Los dossiers en `contenido/` son la investigación completa por bloque. Cada uno incluye el material conceptual, datos concretos de abril 2026, ejemplos de código y **sus propias fuentes al final**. Extensión 2500-4500 palabras por archivo.

- `contenido/01-fundamentos.md` — Fundamentos LLM y panorama de modelos frontier.
- `contenido/02-tokens-contexto.md` — Tokenización, ventanas de contexto, costes, prompt caching.
- `contenido/03-proveedores.md` — Mapa del ecosistema, pricing, self-hosting, compliance.
- `contenido/04-frameworks.md` — Frameworks (LangChain, LlamaIndex, etc.), MCP, debate framework vs SDK.
- `contenido/05-apis.md` — APIs de chat completions, estándar OpenAI-compatible, streaming, tool use, structured output.

## Bibliografía y referencias

- [referencias.md](referencias.md) — Papers fundacionales, docs oficiales, frameworks, cursos, libros, blogs, benchmarks, herramientas, comunidades.

---

## Referencias del curso

- **Proyecto EdTech**: `/Users/manduinca/Projects/deepskill/educa-AI-Platforms` — orquestación LLM, chat multi-turn, streaming SSE, exports, rate limiting.
- **Proyecto electoral**: `/Users/manduinca/Projects/deepskill/peru-elecciones` — Go + DeepSeek vía API OpenAI-compatible, contexto SQLite sin RAG. Caso "minimal viable LLM app".

---

## Checklist previo a impartir (verificar frescura)

> Los dossiers marcan con `[V]` / ⚠️ los datos volátiles. Revisar estas URLs el mismo día de la clase:

- [ ] Modelos actuales de Anthropic: https://docs.anthropic.com/en/release-notes/overview
- [ ] Modelos OpenAI: https://platform.openai.com/docs/models
- [ ] Gemini: https://ai.google.dev/gemini-api/docs/models
- [ ] LMArena top-10 del día: https://lmarena.ai
- [ ] Artificial Analysis pricing/latencia: https://artificialanalysis.ai
- [ ] Precios Anthropic / OpenAI / Gemini / DeepSeek (cambian cada 6-8 semanas).
- [ ] Estado de ARC-AGI-2 y HLE.

---

## Próximos pasos sugeridos

1. **Validar estructura con un revisor** (colega o AI) antes de pasar a slides.
2. **Convertir a slides**: recomiendo 25-30 slides para 90 min (1 slide cada 3 min de mediana). El dossier tiene ~15-20k palabras de material; filtrar a lo esencial.
3. **Grabar los demos antes de la clase** como fallback si la conexión falla.
4. **Preparar notebook Python con los 3 demos** (tokenización, llamada dual, needle-in-haystack).

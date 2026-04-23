# Clase 3 — RAG (*retrieval augmented generation*)

> Clase 3 (de 5) del curso *Arquitectura de Aplicaciones con IA Generativa* — EscuelaIT.
> Duración: 75 min. Nivel: desarrolladores intermedio-avanzados.

---

## Objetivo

Al terminar la sesión, el asistente debe ser capaz de:

1. Distinguir **prompting**, **RAG** y **fine-tuning** como respuestas arquitectónicas distintas, y decidir cuándo usar cada una en función de coste, frescura de datos y tipo de tarea.
2. Explicar qué es un **embedding**, cómo se entrena (contrastivo), y por qué la **similitud coseno** es la medida canónica para recuperar por significado.
3. Describir la anatomía de un sistema RAG — *chunking*, *vector store*, *retrieval*, *augmented generation*, *citaciones*.
4. Implementar un RAG mínimo funcional con SQLite puro y `sentence-transformers`, entendiendo cada paso sin cajas negras.
5. Reconocer estrategias de **chunking** y **retrieval** de producción (híbrido, reranking, HyDE), los modos de fallo típicos y los casos donde RAG sobra.

---

## Estructura (75 min)

| # | Bloque | Min | Contenido | Dónde |
|---|---|---|---|---|
| 0 | **Apertura + puente desde clase 2** | 3 | Cierre clase 2 → ¿qué pasa cuando el conocimiento no está en el modelo? | `01-conceptos.ipynb` |
| 1 | **Cuatro límites del LLM puro** | 4 | Cutoff, privacidad, alucinaciones, ventana de contexto. | `01-conceptos.ipynb` §1 |
| 2 | **Prompting · RAG · Fine-tuning** | 8 | Tabla comparativa. Heurística de decisión. Antipattern del fine-tuning. | `01-conceptos.ipynb` §2 |
| 3 | **Costes y casos de uso** | 4 | Orden de magnitud, dónde cae natural cada patrón. | `01-conceptos.ipynb` §3 |
| 4 | **Embeddings** | 8 | Qué son, entrenamiento contrastivo, densos vs dispersos, dimensionalidad. | `01-conceptos.ipynb` §4 |
| 5 | **Similitud coseno** | 4 | Fórmula, interpretación, coseno vs euclidiana, el truco de normalizar. | `01-conceptos.ipynb` §5 |
| 6 | **Anatomía de un RAG + glosario** | 5 | Ingesta y consulta (diagramas), cinco conceptos clave. | `01-conceptos.ipynb` §6 |
| 7 | **Demo — Setup + corpus + chunking** | 7 | Venv, deps, corpus `.txt`, función de chunking. | `02-demo-rag.ipynb` §0–1 |
| 8 | **Demo — Embeddings con MiniLM** | 10 | Cargar modelo, ver vector, embeber corpus. | `02-demo-rag.ipynb` §2 |
| 9 | **Demo — Vector store en SQLite + retrieval** | 10 | BLOB, SQL plano, coseno con NumPy, top-K. | `02-demo-rag.ipynb` §3–4 |
| 10 | **Demo — Generación con vs sin RAG** | 7 | Comparación lado a lado. Bonus: `sqlite-vec` como versión producción. | `02-demo-rag.ipynb` §5–6 |
| 11 | **Chunking y retrieval en producción** | 4 | Estrategias (recursive, semantic), K, híbrido, reranking, HyDE. | `01-conceptos.ipynb` §7–8 |
| 12 | **Modos de fallo + cuándo NO usar RAG** | 3 | Fallos típicos y regla mental de decisión. | `01-conceptos.ipynb` §9–10 |
| 13 | **Cierre + preview clase 4** | 2 | Tres take-aways, transición a construcción de aplicación. | `01-conceptos.ipynb` |

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
jupyter lab clase-3/notebook/
```

La primera ejecución de la demo descarga el modelo de embeddings MiniLM (~420 MB) desde HuggingFace a `~/.cache/huggingface/`. Hazlo antes de la clase.

---

## Tres mensajes que deben llevarse

1. **RAG no es magia: es búsqueda semántica + inyección de contexto en el prompt.** Todo lo demás son detalles de ingeniería (chunking, vector store, reranking).
2. **La elección entre prompting, RAG y fine-tuning es arquitectónica, no técnica.** Depende de cuánto cambia el conocimiento, cuánto cuesta cada llamada y qué tipo de tarea resuelves.
3. **Lo que construimos hoy vive en los sistemas reales.** El núcleo de antapaccay-demo es este mismo patrón más retrieval híbrido, reranking y citaciones.

---

## Conexión con otras clases

- **Viene de:** clase 2 (prompts como código, pipelines multi-etapa, output estructurado).
- **Lleva a:** clase 4 (construcción de una aplicación LLM completa: frontend, backend, persistencia, streaming).

---

## Proyecto de referencia

El stack de la demo (SQLite + `sentence-transformers` + LLM agnóstico) es una versión didáctica simplificada del que usa `/Users/manduinca/Projects/deepskill/antapaccay-demo` en producción. Allí se añaden: extensión `sqlite-vec` para búsqueda vectorial nativa, BM25 + RRF para retrieval híbrido, y `pymupdf4llm` para ingesta de PDFs.

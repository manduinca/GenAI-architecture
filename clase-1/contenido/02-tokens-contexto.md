# Bloque 2 — Tokenización, ventanas de contexto y límites

> Dossier de investigación para la **Clase 1** del curso *Arquitectura de Aplicaciones con IA Generativa* (EscuelaIT, abril 2026). Nivel: desarrolladores intermedio-avanzados.
>
> **Nota metodológica:** Los precios y context windows de la Sección 5 deben reverificarse contra las páginas oficiales (anthropic.com/pricing, openai.com/api/pricing, ai.google.dev/pricing, artificialanalysis.ai) antes de impartir. Celdas marcadas `[V]` son las más volátiles.

---

## 1. ¿Qué es un token? (explicación técnica para developers)

Un **token** es la unidad atómica de texto que un LLM procesa. No es un carácter, no es una palabra: es un fragmento de texto —típicamente entre 1 y ~10 caracteres— que el tokenizador del modelo ha aprendido a tratar como un "símbolo" indivisible. El modelo nunca ve caracteres ni palabras; solo ve **IDs enteros** que mapean a entradas de un vocabulario finito (típicamente entre 50.000 y 200.000 entradas).

### ¿Por qué no son palabras?

Dividir por espacios falla para idiomas sin separadores (chino, japonés), produce vocabularios enormes (millones de palabras flexionadas en español), y no generaliza a palabras nuevas ("chatgptear", "vibecoding"). Dividir por caracteres soluciona el vocabulario pero estira las secuencias 4-5x, lo que es carísimo en un Transformer (cuyo cómputo de atención crece O(n²) con la longitud). Los **subword tokenizers** son el compromiso: piezas frecuentes se codifican en 1 token; piezas raras se descomponen en varios tokens más cortos.

### Reglas prácticas ("back-of-the-envelope")

- **Inglés:** ~4 caracteres por token / ~0,75 tokens por palabra → 1.000 tokens ≈ 750 palabras (regla oficial de OpenAI).
- **Español:** ~2,5-3 caracteres por token / ~1,3-1,5 tokens por palabra. Un texto en español consume **~1,3-1,5x más tokens** que el mismo texto en inglés.
- **Chino/japonés/coreano:** típicamente 1-2 caracteres por token, pero el conteo absoluto de tokens por "idea" puede ser 2-3x el del inglés.
- **Código:** muy variable; Python denso ~3,5 char/token, JSON con comillas y llaves puede bajar a 2-2,5 char/token.

### Diferencia conceptual

| Unidad | Ejemplo "tokenización" | Conteo |
|---|---|---|
| Caracteres | t·o·k·e·n·i·z·a·c·i·ó·n | 12 |
| Palabras | "tokenización" | 1 |
| Tokens (cl100k) | "token" + "ización" | 2 |
| Tokens (o200k) | "tokenización" | 1 (si frecuente) |

---

## 2. Algoritmos de tokenización

### 2.1 BPE (Byte-Pair Encoding)

Algoritmo propuesto por Sennrich et al. (2016) adaptado de compresión. Parte de bytes individuales y **fusiona iterativamente el par más frecuente** hasta alcanzar un vocabulario objetivo. Es determinista, reversible y —en su variante byte-level— no tiene token "desconocido" porque cualquier byte UTF-8 es representable.

- **Usado por:** GPT-2/3/4/5, Claude (Anthropic usa su propia variante BPE byte-level), LLaMA (BPE sobre SentencePiece), Mistral, la mayoría de los modelos frontier actuales.
- **Ventaja clave:** robusto a cualquier entrada (incluido código, emojis, bytes inválidos).

### 2.2 tiktoken (OpenAI)

`tiktoken` es la librería open-source que implementa los tokenizadores de OpenAI en Rust+Python. Encodings principales:

| Encoding | Vocab size | Modelos |
|---|---|---|
| `cl100k_base` | ~100.256 | GPT-3.5-turbo, GPT-4, GPT-4-turbo, text-embedding-3-* |
| `o200k_base` | ~200.019 | GPT-4o, GPT-4.1, GPT-5, o-series |
| `p50k_base` / `r50k_base` | ~50K | Legacy (Codex, davinci) |

`o200k_base` duplicó el vocabulario respecto a `cl100k` y **mejora 1,3-1,7x la eficiencia en idiomas no-ingleses** (según el blog de GPT-4o). Para español, el ahorro típico es ~15-25% de tokens.

```python
import tiktoken
enc = tiktoken.encoding_for_model("gpt-5")  # → o200k_base
tokens = enc.encode("Hola, ¿cómo estás?")
print(len(tokens), tokens)
```

### 2.3 SentencePiece

Librería de Google (Kudo & Richardson, 2018) que implementa BPE **y** Unigram LM sobre texto crudo, sin pre-tokenización por espacios. Trata el espacio como `▁` (U+2581) y lo incluye en los tokens → reversibilidad perfecta.

- **Usado por:** LLaMA 1/2/3/4 (BPE-mode), Gemini, T5, mT5, PaLM, la mayoría de modelos de Google.
- **Ventaja:** agnóstico de idioma (no asume espacios → bueno para japonés/chino/tailandés).

### 2.4 WordPiece

Variante de BPE de Google (Schuster & Nakajima, 2012) que en lugar de fusionar por frecuencia fusiona por **máxima verosimilitud** (score = freq(AB) / freq(A)·freq(B)). Marca continuaciones de palabra con `##`.

- **Usado por:** BERT, DistilBERT, ELECTRA. Prácticamente no se usa en LLMs generativos modernos.

### 2.5 ¿Cómo saber qué tokenizador usa un modelo?

| Modelo | Tokenizador | Cómo contar |
|---|---|---|
| GPT-3.5/4 | cl100k_base | `tiktoken.encoding_for_model(...)` |
| GPT-4o/4.1/5 | o200k_base | `tiktoken.encoding_for_model(...)` |
| Claude 3/4.x | BPE propio (Anthropic) | `client.messages.count_tokens(...)` en SDK oficial |
| Gemini | SentencePiece | `genai.GenerativeModel(...).count_tokens(...)` |
| LLaMA 3/4 | Tiktoken-compat BPE (OAI) o SP | `transformers.AutoTokenizer.from_pretrained(...)` |
| Mistral | SentencePiece BPE | `mistralai.client.tokenize` / HF |
| DeepSeek | BPE propio (HF) | `transformers.AutoTokenizer` |
| Qwen | BPE propio tipo tiktoken | HF tokenizer |

### 2.6 Herramientas para contar tokens

- **OpenAI:** `tiktoken` (Python/JS) + tokenizer web en `platform.openai.com/tokenizer`.
- **Anthropic:** SDK oficial expone `client.messages.count_tokens(model=..., messages=...)` — importante porque no hay un tokenizador público idéntico al de producción.
- **Google:** `count_tokens()` en el SDK de `google-genai`.
- **HuggingFace:** `AutoTokenizer.from_pretrained("meta-llama/Llama-3.1-70B")` → funciona para todos los modelos abiertos.
- **Tiktokenizer** (web, de dqbd): `tiktokenizer.vercel.app` — excelente para demos en vivo porque pinta los tokens con colores.

---

## 3. Consecuencias prácticas de la tokenización

### 3.1 El coste se paga por token, no por palabra

La factura de cualquier API LLM es `(tokens_input × precio_in + tokens_output × precio_out)`. Esto implica tres cosas no-obvias:

1. **Resumir no siempre ahorra dinero** si el resumen introduce vocabulario infrecuente que tokeniza peor que el original.
2. **Formato estructurado (JSON, XML) infla el coste** versus texto plano: un JSON con comillas, llaves y claves repetidas puede costar 1,3-1,8x el equivalente en Markdown.
3. **El prompt del sistema se paga cada turno** salvo que uses prompt caching (ver 7.4).

### 3.2 El "language tax": idiomas no-ingleses pagan más

Petrov et al. ("Language Model Tokenizers Introduce Unfairness Between Languages", NeurIPS 2023) midieron que la misma frase en distintos idiomas cuesta muy diferente número de tokens:

| Idioma | Tokens para la misma frase (normalizado vs inglés) |
|---|---|
| Inglés | 1,0x |
| Español / Francés / Alemán | 1,3-1,5x |
| Ruso / Árabe | 2,5-3x |
| Hindi / Tamil | 5-7x |
| Birmano / Shan | 10-15x |

Con `o200k_base` (GPT-4o en adelante) el gap se **redujo ~30%** pero no desaparece. Para una plataforma que sirve contenido en español (ej. `peru-elecciones`, EscuelaIT), eso significa que **cada conversación cuesta ~40% más** que una equivalente en inglés, y consumes la ventana de contexto más rápido.

### 3.3 Números, código y JSON tokenizan raro

- **Números:** `cl100k_base` tokeniza "1234" como 1 token, pero "12345" como 2 tokens ("123" + "45"). Esto rompe razonamiento aritmético y explica por qué los modelos fallan más con números de ≥4 dígitos. `o200k_base` agrupa dígitos de 3 en 3 de forma más consistente.
- **Código Python:** identificadores comunes (`def`, `self`, `return`) son 1 token; nombres largos en camelCase se parten (`getUserById` → `get`, `User`, `By`, `Id`).
- **Whitespace:** en `cl100k_base` cada combinación de espacios al inicio de línea es un token propio ("    ", "        "), lo que hace que el código Python tokenice 20-30% más eficiente que JavaScript con tabs mezclados.
- **JSON:** comillas dobles, comas y llaves añaden overhead; claves repetidas (`"id":`, `"name":`) no se comprimen entre entradas. Regla práctica: 100 registros JSON ≈ 150 registros equivalentes en CSV.

### 3.4 Glitch tokens: "SolidGoldMagikarp" y amigos

En febrero de 2023, Rumbelow y Watkins (LessWrong) descubrieron que el tokenizador de GPT-2/3 contenía ~140 tokens "huérfanos" (`SolidGoldMagikarp`, `StreamerBot`, ` petertodd`, `davidjl`) que existían en el vocabulario pero nunca aparecían en los datos de entrenamiento — resultado de que el vocabulario BPE se entrenó sobre un corpus distinto al de pre-training. Al invocarlos, el modelo entraba en estados anómalos: alucinaba, insultaba, se negaba a repetir la palabra.

Moraleja técnica: **el tokenizador es un artefacto separado del modelo** y su entrenamiento independiente puede generar patologías. OpenAI corrigió la mayoría en `cl100k_base` y casi todos en `o200k_base`, pero el fenómeno sigue vivo en modelos open-source que heredan vocabularios antiguos.

---

## 4. Ventanas de contexto

Técnicamente, la **ventana de contexto** es la longitud máxima de la secuencia de tokens que el modelo puede procesar en un forward pass. Está determinada por:

1. La **matriz de embeddings posicionales** (RoPE, ALiBi) entrenada hasta cierta longitud.
2. El **cuello de botella KV-cache en memoria GPU** (O(n) por capa, por cabeza).
3. El **coste O(n²) de la atención** — por eso modelos con 1M+ usan atención sparse, ring attention, o sliding window (Mistral).

Incluye **todo**: system prompt + historia de chat + documentos inyectados + output generado. El límite de "max output tokens" suele ser muy inferior al total (8K-64K típicamente).

### 4.1 Evolución histórica

| Año | Modelo representativo | Context window |
|---|---|---|
| 2018 | BERT, GPT-1 | 512 |
| 2020 | GPT-3 | 2.048 |
| 2022 | GPT-3.5, ChatGPT | 4.096 |
| 2023 | GPT-4, Claude 2 | 8K-100K |
| 2023 (fin) | GPT-4-turbo | 128K |
| 2024 | Gemini 1.5 Pro | 1M (2M preview) |
| 2024 | Claude 3.5 Sonnet | 200K |
| 2024 | Magic.dev LTM-2 | 100M (experimental) |
| 2025 | Gemini 2.5 Pro | 1M (2M en Vertex) |
| 2025 | GPT-5 | 400K-1M segmentado |
| 2025-26 | Llama 4 Scout | 10M (declarado) |
| 2026 | Claude Opus 4.7 (1M context) | 1M |

El salto de 4K (2022) a 1M (2024) son **~8 duplicaciones en 24 meses**, pero el crecimiento se está estabilizando porque el cuello de botella ya no es poder procesar tokens, sino **usarlos bien** (ver Sección 6).

---

## 5. Tabla de context windows y precios (abril 2026) `[VERIFICAR]`

> Los precios están en USD por 1M de tokens. Dada la volatilidad del mercado, **verificar en las URLs oficiales antes de la clase**. Los marcados con `[V]` son los que más cambian.

### Anthropic — Claude

| Modelo | Context | Max output | Input $/MTok | Output $/MTok | Cache write | Cache read (hit) |
|---|---|---|---|---|---|---|
| Claude Opus 4.7 (1M) | 1.000.000 | 64K | ~15,00 `[V]` | ~75,00 `[V]` | ~18,75 (5m) / ~30 (1h) | ~1,50 |
| Claude Sonnet 4.6 | 200K / 1M | 64K | 3,00 (≤200K) / 6,00 (>200K) | 15 / 22,50 | 3,75 / 7,50 | 0,30 / 0,60 |
| Claude Haiku 4.5 | 200K | 8K | 1,00 | 5,00 | 1,25 | 0,10 |

*Prompt caching de Anthropic:* escribes al precio de cache-write (1,25x input) y cada hit cuesta 0,1x del precio input. TTL 5 minutos por defecto, 1 hora en tier extendido (2x write).

### OpenAI — GPT-5 family

| Modelo | Context | Max output | Input $/MTok | Cached input | Output $/MTok |
|---|---|---|---|---|---|
| GPT-5 | 400K-1M | 128K | ~1,25-2,50 `[V]` | ~0,125-0,25 | ~10,00 `[V]` |
| GPT-5 mini | 400K | 128K | ~0,25 | ~0,025 | ~2,00 |
| GPT-5 nano | 400K | 64K | ~0,05 | ~0,005 | ~0,40 |
| o4 / o-series reasoning | 200K-400K | 100K+ | ~3,00-15,00 `[V]` | ~0,30-1,50 | ~12,00-60,00 |
| GPT-4.1 | 1M | 32K | ~2,00 | ~0,50 | ~8,00 |

*Prompt caching OpenAI:* automático desde 2024; los tokens cacheados cuestan 10-25% del precio input, sin API adicional (se activa a partir de 1024 tokens idénticos en el prefijo).

### Google — Gemini

| Modelo | Context | Max output | Input $/MTok (≤200K) | Input (>200K) | Output $/MTok |
|---|---|---|---|---|---|
| Gemini 3 Pro / 2.5 Pro | 1M (2M Vertex) | 64K | ~1,25 `[V]` | ~2,50 | ~10,00 / ~15,00 |
| Gemini 3 Flash / 2.5 Flash | 1M | 64K | ~0,30 | ~0,60 | ~2,50 |
| Gemini Flash-Lite | 1M | 8K | ~0,075 | ~0,15 | ~0,30 |

*Context caching Gemini:* storage cost $1,00/MTok/hora + descuento ~75% sobre hits. Excelente para vídeo largo cacheado.

### Open weights / otros

| Modelo | Context | Input $/MTok (via API host) | Output $/MTok |
|---|---|---|---|
| DeepSeek V3.5 / R2 | 128K-256K | ~0,14-0,27 (off-peak 50% desc.) `[V]` | ~1,10-2,20 |
| Llama 4 Scout | 10M (declarado) | ~0,10-0,30 (Groq/Together/Fireworks) | ~0,40-0,80 |
| Llama 4 Maverick | 1M | ~0,50-1,00 | ~1,50-3,00 |
| Qwen 3 Max | 256K-1M | ~0,30-0,80 `[V]` | ~1,20-3,20 |
| Mistral Large 3 | 256K | ~2,00 | ~6,00 |
| Mistral Small 3 | 128K | ~0,20 | ~0,60 |

Para comparativas live: **artificialanalysis.ai** mantiene una tabla actualizada con precio mediano ponderado por input/output 3:1, latencia p50 y tokens/s.

---

## 6. Límites reales vs. nominales

Que un modelo **acepte** 1M tokens en la API no significa que los **use bien**. Tres benchmarks críticos:

### 6.1 Needle-in-a-Haystack (NIAH)

Greg Kamradt popularizó el test en 2023: se inserta una "aguja" ("The best thing to do in San Francisco is eat a sandwich at Dolores Park on a sunny day") en un montón creciente de texto y se pregunta por ella. GPT-4 (128K) y Claude 2.1 mostraron **recall >95% hasta ~70K** y caídas fuertes hacia el final de la ventana. Hoy, Gemini 2.5 Pro, Claude Opus 4.x y GPT-5 reportan >99% en NIAH clásico hasta 1M.

**Pero NIAH es demasiado fácil.** Un solo hecho atómico, sin distractores semánticamente similares. Pasar NIAH ≠ saber usar 1M.

### 6.2 Lost in the Middle (Liu et al., 2023)

Nelson Liu y colegas (Stanford) mostraron que los modelos recuerdan mucho mejor información al **inicio** y al **final** del contexto que en el medio — un "U-shape" clásico. La precisión en QA con 20 documentos cae del 75% (documento relevante en posición 1) al ~50% (posición 10) y sube de nuevo al ~65% (posición 20). Paper: arxiv.org/abs/2307.03172.

Implicación práctica: en un prompt largo, **pon la instrucción crítica al principio Y al final**, y coloca el documento más relevante cerca de los extremos.

### 6.3 RULER (NVIDIA, 2024) y LongBench v2

RULER (Hsieh et al., arxiv.org/abs/2404.06654) extiende NIAH con 13 tareas: multi-needle, tracking de variables, aggregation, QA multi-hop. Resultados típicos:

- Modelos "1M" frontier: **effective context ~128K-400K** (donde mantienen >85% del score a 4K).
- A 1M tokens literales, la mayoría baja a 60-75% de su rendimiento en corto.

**LongBench v2** (Bai et al., THUDM) añade razonamiento realista sobre 8K-2M tokens; sigue sin haber ningún modelo que mantenga >80% a 1M.

### 6.4 Conclusión operativa

El **effective context** es típicamente 25-50% del nominal. Si te venden 1M, diseña pensando en ~250-500K útiles y baja de ahí si tu tarea exige multi-hop reasoning.

---

## 7. Estrategias de gestión de contexto

### 7.1 Truncamiento

- **FIFO/sliding window:** descartar los turnos más antiguos cuando se supera un budget. Simple pero destruye continuidad.
- **Token budget:** definir `max_context = model_limit - max_output - safety_margin` (ej. 180K en Sonnet para dejar 20K de output). Contar tokens **antes** de enviar, no confiar en que la API truncará bien.
- **Selectivo:** conservar system prompt + últimos N turnos + turnos "anclados" (marcados por el usuario o por relevancia).

### 7.2 Resumen incremental (rolling summary)

Patrón: cada K turnos, resumir la historia previa con el mismo LLM o uno más barato (Haiku/Flash) y sustituir los mensajes por el resumen. Variantes:

- **Hierarchical summary** (LangChain `ConversationSummaryBufferMemory`): resumen + ventana reciente cruda.
- **Entity memory:** extraer entidades mencionadas y mantener un "profile" separado.
- Riesgo: **drift semántico acumulativo**; cada resumen pierde ~10-20% de información fina.

### 7.3 RAG como alternativa a contexto largo

Si tu corpus es >1M tokens o cambia frecuentemente, RAG casi siempre gana a "meter todo en contexto":

- Coste: recuperar 5K tokens relevantes vs. cargar 500K → 100x más barato.
- Latencia: 500K tokens en Claude ≈ 30-60s de TTFT; RAG ≈ 2-5s.
- Calidad: si tu retrieval es bueno (BM25 + dense + rerank), **supera** al long-context en la mayoría de benchmarks de QA (ver RAG vs LC en Gemini 1.5 paper, 2024).

Long-context gana cuando: (a) la tarea requiere **razonamiento cruzado global** sobre todo el corpus, (b) no sabes a priori qué buscar, (c) el corpus es multimodal complejo (vídeo).

### 7.4 Prompt caching

**Anthropic:** marcas bloques con `cache_control: {type: "ephemeral"}`. Escribir cuesta 1,25x input normal; leer cuesta 0,1x. Break-even tras ~3 hits. TTL 5 minutos (renovable) o 1 hora (2x write). Ideal para: system prompts grandes, few-shot extensos, documentos reutilizados entre turnos.

**OpenAI:** desde 2024 es **automático**: si tu prefijo ≥1024 tokens coincide con una petición reciente, los tokens cacheados facturan a ~10-25% del precio input, sin API extra. Cero trabajo, pero menos control.

**Gemini context caching:** API explícita (`caches.create(...)`); paga storage por hora + hits con descuento ~75%. Pensado para documentos/vídeos muy grandes servidos muchas veces.

**Regla de decisión:** si el mismo prefijo se reutiliza ≥3 veces en ≤5 min → cachéalo.

### 7.5 Contexto estructurado (patrón `BuildContext`)

Patrón observado en proyectos de producción (p.ej. `peru-elecciones`, `educa-AI-Platforms`): en lugar de lanzar al prompt todo el estado de conversación, se mantiene una **base de datos (SQLite/Postgres) como fuente de verdad** y una función `buildContext(userId, intent)` que **compone dinámicamente** el prompt con solo los slots relevantes para el turno:

```
[system: instrucciones base]
[perfil_usuario: nombre, idioma, nivel]      ← query a `users`
[hechos_verificados_recientes: top 5]         ← query a `facts` WHERE relevance_score > X
[ultimos_3_turnos]                            ← query a `messages` LIMIT 3
[user: pregunta]
```

Ventajas: el contexto es **determinista, auditable, versionable**. Puedes A/B-testear prompts cambiando solo el builder. Escalas a millones de usuarios sin inflar la ventana.

---

## 8. Coste real de la ventana de contexto (ejemplos numéricos)

### 8.1 Procesar 1M tokens de input una sola vez (sin caching)

| Proveedor | Modelo | Coste input 1M | Coste output 10K (respuesta típica) | Total |
|---|---|---|---|---|
| Anthropic | Opus 4.7 | $15,00 | $0,75 | **$15,75** |
| Anthropic | Sonnet 4.6 (tier >200K) | $6,00 | $0,225 | **$6,23** |
| OpenAI | GPT-5 | ~$2,50 | ~$0,10 | **~$2,60** |
| Google | Gemini 3 Pro (tier >200K) | ~$2,50 | ~$0,15 | **~$2,65** |
| DeepSeek | V3.5 | ~$0,27 | ~$0,022 | **~$0,29** |

Diferencia **~55x** entre el más caro (Opus) y el más barato (DeepSeek) para el mismo trabajo. Pero Opus probablemente razona 2-3x mejor sobre ese contexto → decisión no trivial.

### 8.2 Conversación de 20 turnos con 50K de system prompt + documentos

Supuesto: 50K tokens fijos (system+docs) + ~500 tokens usuario/turno + ~1.500 tokens respuesta/turno. Total acumulado: ~51K tokens leídos en turno 1, creciendo hasta ~91K en turno 20. **Cada turno re-procesa TODO el contexto previo** (las APIs son stateless).

**Sin caching (Sonnet 4.6):**
- Input total sumado a lo largo de 20 turnos: Σ ≈ 20 × 70K promedio = 1,4M tokens → 1,4 × $3 = **$4,20**
- Output total: 20 × 1.500 × $15/M = **$0,45**
- **Total: $4,65**

**Con prompt caching (Anthropic, TTL 5m, cacheando los 50K fijos):**
- Escritura del caché turno 1: 50K × $3,75 = $0,1875
- Lectura cacheada turnos 2-20 (19 hits): 19 × 50K × $0,30 = $0,285
- Tokens dinámicos (no cacheados): ~20 × 10K × $3 = $0,60
- Output: $0,45
- **Total: ~$1,52 → ahorro ~67%**

### 8.3 Regla de bolsillo para presupuestar

`coste_USD ≈ (turnos × tokens_fijos × precio_in × (0,1 si cache else 1)) + (Σ turnos_dinámicos × precio_in) + (turnos × tokens_out × precio_out)`

El primer término es el que explota sin caching; el tercero el que explota si permites respuestas largas sin necesidad.

---

## 9. Ideas para demos en vivo (3 ideas listas para usar en 90 min)

### Demo 1 — Tokenización comparada entre idiomas (10 min)

**Objetivo:** visualizar el "language tax" y la diferencia entre encodings.

```python
import tiktoken

textos = {
    "es": "La inteligencia artificial transformará la economía global en la próxima década.",
    "en": "Artificial intelligence will transform the global economy in the next decade.",
    "zh": "人工智能将在未来十年改变全球经济。",
    "code": "def fibonacci(n):\n    return n if n<2 else fibonacci(n-1)+fibonacci(n-2)",
}

for enc_name in ["cl100k_base", "o200k_base"]:
    enc = tiktoken.get_encoding(enc_name)
    print(f"\n=== {enc_name} ===")
    for lang, t in textos.items():
        toks = enc.encode(t)
        print(f"{lang:5} | chars={len(t):4} | tokens={len(toks):4} | ratio={len(t)/len(toks):.2f}")
```

Proyectar después `tiktokenizer.vercel.app` con los mismos textos para ver los tokens pintados. Subrayar que **la misma frase cuesta 1,4x en español que en inglés**, y que `o200k_base` redujo la brecha ~20%.

### Demo 2 — Comparativa de coste real entre 3 proveedores (10 min)

Un script que manda el mismo prompt (un PDF de ~30K tokens + pregunta) a Claude Sonnet, GPT-5 y Gemini Flash y reporta: tokens in/out, latencia TTFT, latencia total, coste USD calculado con la tabla de precios. Repetirlo con **prompt caching activado** para Anthropic y mostrar la reducción del 67%.

Punto pedagógico: **no hay un "ganador" absoluto**. Haiku/Flash pueden ser 30x más baratos; Opus/GPT-5 razonan mejor en tareas multi-hop. La elección es un trade-off de coste × latencia × calidad por tarea.

### Demo 3 — Degradación needle-in-haystack al crecer el contexto (15 min)

Usar el repo open-source de Greg Kamradt (`LLMTest_NeedleInAHaystack`) o una versión propia: insertar la frase "La contraseña secreta del curso EscuelaIT 2026 es `bl0que2`" en posiciones {10%, 50%, 90%} de un documento sintético de tamaño variable {10K, 50K, 200K, 500K, 1M}. Preguntar "¿Cuál es la contraseña secreta?" y graficar recall.

Incluso con modelos que declaran 1M, la curva típica muestra caídas al 70-85% a partir de ~200K. Conectar con la Sección 6: **ventana nominal ≠ ventana efectiva**. Cerrar con el mensaje clave: "más contexto no es gratis, ni en dinero ni en calidad".

---

## Fuentes

### Pricing y docs oficiales
- Anthropic pricing: https://www.anthropic.com/pricing
- Anthropic Claude models overview: https://docs.anthropic.com/en/docs/about-claude/models/overview
- Anthropic prompt caching: https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- Anthropic token counting: https://docs.anthropic.com/en/api/messages-count-tokens
- OpenAI pricing: https://openai.com/api/pricing/
- OpenAI tokenizer (web): https://platform.openai.com/tokenizer
- OpenAI prompt caching: https://platform.openai.com/docs/guides/prompt-caching
- Google Gemini pricing: https://ai.google.dev/gemini-api/docs/pricing
- Google context caching: https://ai.google.dev/gemini-api/docs/caching
- DeepSeek API pricing: https://api-docs.deepseek.com/quick_start/pricing
- Mistral pricing: https://mistral.ai/technology/#pricing
- Together AI (Llama, Qwen): https://www.together.ai/pricing

### Agregadores y benchmarks
- Artificial Analysis (pricing/latencia live): https://artificialanalysis.ai/
- LMSYS Chatbot Arena: https://lmarena.ai/
- HELM (Stanford): https://crfm.stanford.edu/helm/

### Tokenización
- tiktoken (GitHub): https://github.com/openai/tiktoken
- SentencePiece (Google): https://github.com/google/sentencepiece
- HuggingFace tokenizers: https://huggingface.co/docs/tokenizers
- Tiktokenizer (web, dqbd): https://tiktokenizer.vercel.app/
- Sennrich et al., "Neural Machine Translation of Rare Words with Subword Units" (BPE): https://arxiv.org/abs/1508.07909
- Kudo & Richardson, "SentencePiece" (EMNLP 2018): https://arxiv.org/abs/1808.06226
- Petrov et al., "Language Model Tokenizers Introduce Unfairness Between Languages" (NeurIPS 2023): https://arxiv.org/abs/2305.15425
- "SolidGoldMagikarp" (LessWrong, Rumbelow & Watkins, 2023): https://www.lesswrong.com/posts/aPeJE8bSo6rAFoLqg/solidgoldmagikarp-plus-prompt-generation

### Context window y degradación
- Liu et al., "Lost in the Middle" (TACL 2023): https://arxiv.org/abs/2307.03172
- Hsieh et al., "RULER: What's the Real Context Size of Your Long-Context Language Models?" (2024): https://arxiv.org/abs/2404.06654
- Greg Kamradt, "Needle in a Haystack" (repo): https://github.com/gkamradt/LLMTest_NeedleInAHaystack
- Bai et al., "LongBench v2": https://github.com/THUDM/LongBench
- Gemini 1.5 technical report (long-context vs RAG): https://arxiv.org/abs/2403.05530

---

## Resumen ejecutivo para el slide deck

1. **Tokens ≠ palabras.** ~4 chars/token en inglés, ~2,5-3 en español → español paga ~40% más.
2. **BPE domina** los LLMs generativos (GPT, Claude, Llama, Mistral); SentencePiece en Google; WordPiece solo en BERT.
3. **Idioma, números y JSON** hacen variar el coste 2-5x. Mídelo antes de prometerle al cliente un precio por conversación.
4. **1M tokens nominales ≠ 1M efectivos.** El effective context real es 25-50% del declarado (RULER, Lost in the Middle).
5. **Prompt caching es la palanca de coste #1** en producción: -67% fácil con system prompts largos.
6. **Long-context vs RAG:** RAG gana en coste/latencia; long-context gana en razonamiento global.
7. **Estructura tu contexto** (`buildContext()` desde DB) en lugar de pasar historia cruda.

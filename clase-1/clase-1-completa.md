# Clase 1 — Arquitectura IA Generativa y el Ecosistema LLM

**Curso:** *Arquitectura de Aplicaciones con IA Generativa* — EscuelaIT
**Duración:** 90 min · **Formato:** clase gratuita, remota · **Nivel:** developers intermedio-avanzados
**Fecha:** 17 de abril 2026

---

## Cómo leer este documento

Este es el **guion maestro** de la clase 1. Integra el contenido textual completo de los seis bloques, listo para revisar sección por sección antes de destilarlo a slides LaTeX/Beamer.

Cada bloque abre con su duración estimada y cierra con las transiciones de enlace al siguiente. Las tablas y ejemplos de código están embebidos en la narrativa; no hay que saltar a archivos secundarios. Los datos volátiles (versiones de modelos, precios) están verificados a 17-abr-2026 y marcados donde corresponde.

**Hilo narrativo:** *de ChatGPT-la-caja-mágica al endpoint HTTP que controlamos*. Cada bloque desmitifica una capa del stack: qué es un modelo → cómo le hablamos → quién lo ofrece → cómo lo orquestamos → la API concreta con la que lo invocamos.

**Mensaje central:** el LLM es una capa de infraestructura, no una feature. La diferenciación del producto vive alrededor — prompts, datos, tooling, evaluación, UX. Este curso enseña a construir ese "alrededor".

**Postura pedagógica:** foco en conceptos. Las tecnologías concretas se citan y comparan pero no se profundizan. El alumno sale con un mapa del territorio, no con un manual de LangChain. Las diferencias entre implementaciones son menores; lo que hay que tener claro es el concepto detrás.

---

## Bloque 0 — Apertura · 8 min

### 0.1 Demo gancho (3 min)

Abrimos con una demostración en vivo del asistente del proyecto `peru-elecciones`, una aplicación de producción que consume los datos oficiales de la ONPE (Oficina Nacional de Procesos Electorales del Perú) y permite al usuario hacer preguntas en lenguaje natural sobre los resultados electorales. La demo responde a preguntas como *"¿quién va segundo en Lima?"*, *"¿cuánta participación hay en el norte?"*, *"dame los tres partidos con más crecimiento en las últimas dos horas"*.

El propósito no es presumir la app. El propósito es que en los primeros tres minutos el asistente **reconozca lo que vamos a saber construir**. Es el ancla emocional de toda la sesión: *"al terminar las cinco clases, cada uno de ustedes sabrá construir esto, y mejor que esto"*.

Detalle técnico que se revela sin profundizar: la app usa DeepSeek vía una API OpenAI-compatible, inyecta contexto desde una base SQLite local y no usa RAG. Todo eso cobrará sentido durante el curso.

### 0.2 Evolución del desarrollo de software asistido por IA (4 min)

Un segundo ancla antes de entrar en teoría. Como desarrolladores, **ya somos usuarios** de software que usa IA. Eso que usamos cada día ilumina lo que vamos a construir para nuestros usuarios. Vale la pena trazar la línea del tiempo:

- **2022 — ChatGPT en el navegador.** Pegar código, pedir explicación, copiar de vuelta. Copy-paste-driven development.
- **2023 — GitHub Copilot in-line.** Autocompletado tokens-a-token en el editor. La IA empieza a vivir donde escribimos código.
- **2024 — Cursor, Windsurf.** Chat dentro del IDE con contexto del workspace. Pasamos de autocompletar a conversar con el repositorio.
- **2025 — Claude Code, Codex CLI, Qwen Code, OpenCode.** Agentes en la terminal: leen archivos, corren comandos, editan, se auto-verifican. Ya no autocompletan: **ejecutan tareas**.
- **2026 — IDE como orquestador de agentes.** Zed, Cursor, JetBrains adoptan el **Agent Client Protocol (ACP)**: un agente = un plugin. El IDE se vuelve dashboard; los agentes son intercambiables.

Esta evolución no es anecdótica: mapea las cinco clases del curso. En el curso aprendemos a construir, para nuestros usuarios, el mismo tipo de software que nosotros ya consumimos como desarrolladores. Y el marco para pensarlo es este:

```
┌───────────────────────────────────────────────────────┐
│  SOFTWARE QUE ADMINISTRA                              │
│  tu app · Cursor · Claude Code · peru-elecciones      │
└────────────────────────┬──────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────┐
│  CAPA AGÉNTICA                                        │
│  loops, tools, memoria, MCP, orquestación, guardrails │
└────────────────────────┬──────────────────────────────┘
                         │
┌────────────────────────▼──────────────────────────────┐
│  LLM BASE                                             │
│  Claude, GPT, Gemini, DeepSeek, Qwen, GLM...          │
└───────────────────────────────────────────────────────┘
```

Volvemos a este diagrama varias veces. Es la columna vertebral conceptual del curso entero.

### 0.3 Plano del curso (1 min)

Una slide que muestra los cinco módulos, el foco de cada uno, y dónde cae la clase de hoy. La clase 1 es panorámica — *el mapa del territorio*. Las clases 2-5 entran a construir: prompts como sistema (c.2), patrones arquitectónicos como RAG y orquestación (c.3), construcción de aplicaciones end-to-end (c.4), despliegue y producción (c.5).

**Transición al Bloque 1:** *"Para hablar del territorio, empecemos por lo que hay en el centro: el modelo."*

---

## Bloque 1 — Fundamentos + marco de capas · 10 min

### 1.1 Qué es un LLM

Un **Large Language Model** es una red neuronal profunda — casi siempre con arquitectura Transformer — entrenada sobre cantidades masivas de texto (decenas de terabytes, billones de tokens) con un objetivo autosupervisado: **predecir el siguiente token** dado el anterior. Al final del entrenamiento se obtiene un sistema que modela la distribución de probabilidad del lenguaje humano y que, al muestrearse autoregresivamente, genera texto coherente.

La diferencia con el ML "clásico" es cualitativa, no solo cuantitativa:

| | ML clásico | LLM / fundacional |
|---|---|---|
| Modelo por tarea | uno por tarea (SVM para spam, XGBoost para fraude) | uno para cientos de tareas |
| Cómo se adapta | fine-tuning específico | prompting en lenguaje natural |
| Datos | etiquetados y features diseñadas | texto crudo, autosupervisado |
| Rendimiento al escalar | se satura | sigue leyes de potencia |

El término **"modelo fundacional"** fue acuñado por el Stanford CRFM en 2021 (Bommasani et al.). La idea: son modelos entrenados a gran escala y **adaptables a un rango amplio de tareas downstream**. "Fundacional" no alude al tamaño sino al rol arquitectónico: son los cimientos sobre los que se construyen aplicaciones.

Para el desarrollador esto tiene una consecuencia práctica: **ya no entrenamos modelos**. Los consumimos como capa de infraestructura — vía API o desplegando pesos — y construimos *alrededor*.

### 1.2 El Transformer en una idea

La arquitectura que hizo posible todo esto fue propuesta en 2017 por Vaswani et al. en *Attention Is All You Need*. Para el dev intermedio-avanzado basta con tres conceptos:

1. **Tokenización**: el texto se parte en tokens (subpalabras). Cada token se mapea a un vector denso de miles de dimensiones.
2. **Self-attention**: cada token "mira" a todos los demás del contexto y pesa cuánto importa cada uno. Todas las posiciones se procesan en paralelo — a diferencia de una RNN, donde la información pasa de vecino en vecino.
3. **Autoregresivo**: en inferencia, el modelo predice un token, lo concatena, y repite. Eso es el streaming que vemos en los chatbots.

Analogía útil: *el Transformer es una mesa redonda donde cada palabra escucha a todas las demás antes de hablar*. Paralelismo brutal = entrenable a gran escala en GPU/TPU.

No profundizamos en matemáticas. El alumno solo necesita saber que la arquitectura es paralelizable, que el self-attention es el mecanismo central, y que es autoregresiva. Con eso entiende por qué el modelo tiene ventana de contexto finita y por qué genera token a token.

### 1.3 Chat vs razonador — la distinción más importante de 2025-2026

Hasta 2024 todos los LLMs funcionaban igual: recibes un prompt, el modelo responde en un forward pass. Desde septiembre de 2024 (OpenAI o1) existen **dos clases de modelos** y conviven:

| | Modelo chat | Modelo razonador |
|---|---|---|
| Ejemplos | GPT-4o, Claude Sonnet, Gemini Flash, DeepSeek-chat | o-series, Claude con extended thinking, DeepSeek-reasoner, GLM-5.1, Gemini Deep Think |
| Cómo responde | un forward pass | genera una cadena de razonamiento larga antes de responder |
| Latencia a primer token | 200-500 ms | 30-180 segundos |
| Coste | bajo | alto (tokens de thinking se facturan) |
| Bueno en | conversación, extracción, RAG, clasificación | matemáticas, coding agéntico, planning, puzzles |

Los razonadores escalan **en tiempo de inferencia**: mejoras calidad gastando más cómputo en el momento, sin reentrenar. Es una dimensión nueva de scaling. Para el arquitecto, implica *decisión por tarea*: chatbot liviano → modelo chat; agente que debe refactorizar código con tests → modelo razonador.

En 2026 casi todos los proveedores frontera ofrecen variantes razonadoras: Claude con `thinking`, GPT con `reasoning_effort`, Gemini con `thinking_level`, DeepSeek con `deepseek-reasoner`. Lo que en 2024 era nuevo hoy es estándar.

### 1.4 Tres límites duros que no se pueden ignorar

A pesar del estado del arte, en 2026 los LLMs siguen fallando en cosas concretas. Las tres que más importan para arquitectura:

1. **Alucinación.** Genera información plausible pero falsa con alta confianza. Causa: el objetivo del modelo es plausibilidad, no verdad. Mitigación: RAG, tool use, verificación.
2. **Conocimiento congelado.** No sabe lo ocurrido después de su fecha de corte. Mitigación: tool use con búsqueda web, RAG sobre fuentes frescas.
3. **Cálculo exacto y consistencia.** Falla en aritmética de muchos dígitos, se contradice en conversaciones largas. Mitigación: code interpreter, memoria explícita.

Estos límites no son bugs a corregir. Son **la razón de que exista el resto del curso**. Cada patrón que veremos — RAG en clase 3, tool use en clase 4, evaluación en clase 5 — es una respuesta arquitectónica a uno de estos fallos.

### 1.5 Marco conceptual: tres capas

Formalizamos aquí el diagrama que vimos en la apertura:

- **Capa 3 — LLM base.** El modelo crudo. Recibe tokens, devuelve tokens. Ejemplos: Claude Opus, GPT-5.4, Gemini 3.1 Pro, DeepSeek V3.2, Qwen 3.6-Plus, GLM-5.1.
- **Capa 2 — Capa agéntica.** Lo que rodea al modelo: prompts, memoria, tools, loops de razonamiento, routing, guardrails, observabilidad. Es donde viven los patrones (ReAct, orchestrator-workers, evaluator-optimizer). Aquí también vive MCP.
- **Capa 1 — Software que administra.** La aplicación concreta que consume la capa agéntica: tu producto, Cursor, Claude Code, `peru-elecciones`. Es lo que el usuario final usa.

Esta distinción explica fenómenos que luego veremos con nombre propio. Por ejemplo: **por qué el chat de DeepSeek genera imágenes pero su API no**. Porque el chat es ya capa 2 montada por DeepSeek sobre su propio LLM; la API te da solo la capa 3. La orquestación es tuya. Lo retomamos en el Bloque 3.

**Transición al Bloque 2:** *"Ahora que sabemos qué hay en la capa base, entremos en la unidad mínima con la que trabaja: el token."*

---

## Bloque 2 — Tokens, contexto y límites · 12 min

### 2.1 Qué es un token

Un token es la unidad atómica de texto que el modelo procesa. **No es un carácter, no es una palabra**: es un fragmento de texto — típicamente entre 1 y 10 caracteres — que el tokenizador aprendió a tratar como símbolo indivisible. El modelo nunca ve caracteres ni palabras; ve enteros que mapean a entradas de un vocabulario finito (50K a 200K entradas según el modelo).

Regla práctica back-of-the-envelope:

- **Inglés:** ~4 caracteres por token, ~0,75 tokens por palabra → 1.000 tokens ≈ 750 palabras.
- **Español:** ~2,5-3 caracteres por token. Un texto en español cuesta **30-40% más tokens** que el mismo texto en inglés.
- **Código:** muy variable; Python denso ~3,5 char/token, JSON con comillas y llaves baja a 2-2,5.

Los dos algoritmos de tokenización dominantes son **BPE (Byte-Pair Encoding)** — usado por GPT, Claude, Llama, Mistral — y **SentencePiece** — usado por Gemini y versiones de Llama. Son detalles internos; al desarrollador le importa una cosa: **poder contar tokens antes de enviar**.

Herramientas útiles:

- `tiktoken` (OpenAI, Python/JS).
- `client.messages.count_tokens` en el SDK de Anthropic.
- `AutoTokenizer.from_pretrained(...)` en HuggingFace — sirve para cualquier modelo open-weights.
- La web **tiktokenizer.vercel.app** para visualizar tokenización en vivo. Muy útil como demo.

### 2.2 El "language tax"

La misma frase en distintos idiomas cuesta muy distinto número de tokens. Petrov et al. (NeurIPS 2023) lo cuantificaron:

| Idioma | Tokens relativos vs inglés |
|---|---|
| Inglés | 1,0× |
| Español, francés, alemán | 1,3-1,5× |
| Ruso, árabe | 2,5-3× |
| Hindi, tamil | 5-7× |
| Birmano, shan | 10-15× |

Para una plataforma en español — EscuelaIT, `peru-elecciones` — significa que **cada conversación cuesta ~40% más** que una equivalente en inglés, y la ventana de contexto se consume más rápido. Con `o200k_base` (GPT-4o en adelante) el gap se redujo ~20%, pero no desaparece.

Implicación de producto: si vas a ofrecer un servicio SaaS multilingüe con precio plano por usuario, el coste por usuario varía 3-7× según idioma. Hay que tenerlo en el modelo financiero.

### 2.3 Ventana de contexto: nominal vs efectivo

La **ventana de contexto** es la longitud máxima de secuencia que el modelo puede procesar. Incluye todo: system prompt + historia de chat + documentos inyectados + la propia respuesta generada.

La evolución en cuatro años es espectacular:

| Año | Modelo representativo | Contexto |
|---|---|---|
| 2022 | GPT-3.5 | 4K |
| 2023 | GPT-4-turbo | 128K |
| 2024 | Gemini 1.5 Pro | 1M |
| 2025 | Claude Sonnet tier enterprise | 1M |
| 2026 | Llama 4 Scout | 10M declarado |

Pero aquí está el truco: **que un modelo acepte 1M tokens no significa que los use bien**. Dos fenómenos bien documentados:

- **Lost in the Middle** (Liu et al., 2023): los modelos atienden mejor al inicio y al final del contexto. En el medio, la precisión cae ~20-30%.
- **RULER** (NVIDIA, 2024): la *ventana efectiva* — donde el modelo mantiene >85% del rendimiento corto — suele ser 25-50% de la nominal.

Implicación práctica: si te venden 1M, diseña pensando en 250-500K útiles. Pon las instrucciones críticas al principio *y* al final del prompt. No asumas que "meter todo el corpus dentro" funcione mejor que un RAG bien diseñado.

### 2.4 Prompt caching: la palanca de coste número 1

Un prompt con prefijo estable largo (system prompt, contexto fijo, documentos reutilizados) puede hashearse y su representación intermedia (KV cache) reutilizarse en llamadas futuras. Los cuatro grandes lo implementan, de formas distintas:

- **Anthropic**: explícito. Marcas hasta 4 breakpoints con `cache_control: {type: "ephemeral"}`. TTL 5 min (default) o 1 h. Lectura: 10% del precio input. Escritura: 125%.
- **OpenAI**: automático desde 2024. Prefijos ≥1024 tokens idénticos facturan 10-25% del precio input.
- **Google Gemini**: explícito vía `cachedContents.create`. Storage por hora + 75% de descuento en hits.
- **DeepSeek**: automático en disco. Reporta `prompt_cache_hit_tokens` y `prompt_cache_miss_tokens`.

Ejemplo numérico concreto — conversación de 20 turnos con 50K tokens fijos de system+contexto:

| Escenario | Coste total |
|---|---|
| Sin caching (Claude Sonnet) | ~$4,65 |
| Con caching Anthropic | ~$1,52 |

**Ahorro 67%.** En producción con millones de llamadas al mes, la diferencia es estructural.

Regla del pulgar: si el mismo prefijo se reutiliza ≥3 veces en 5 minutos, cachéalo.

**Transición al Bloque 3:** *"Ya sabemos qué es un modelo y cómo le hablamos. Pregunta siguiente: ¿quién los ofrece, y cómo elegimos entre ellos?"*

---

## Bloque 3 — Proveedores · 22 min

Este es el bloque más extenso de la clase porque es donde más se juega la sensibilidad geopolítica-técnica del momento: frontera US vs frontera china, cerrado vs abierto, API vs self-host, chat vs API.

### 3.1 El mapa del ecosistema (3 min)

El mercado LLM 2026 no es "OpenAI vs el resto". Conviven **seis capas** que el arquitecto debe conocer porque cada una impone un contrato distinto (API, pricing, compliance, latencia):

1. **Labs frontier propietarios** (cerrados, API-first): Anthropic, OpenAI, Google, xAI.
2. **Labs chinos** (frontera de facto con precios disruptivos): DeepSeek, Alibaba, Moonshot, Z.ai, MiniMax, ByteDance, Zhipu.
3. **Open-weights serios** (pesos descargables con licencia usable): Meta (Llama), Mistral, Cohere, AI2.
4. **Infraestructura de inferencia especializada**: Groq, Cerebras, SambaNova, Together, Fireworks.
5. **Hyperscalers**: AWS Bedrock, Azure AI Foundry, Google Vertex, Oracle OCI.
6. **Self-hosting local**: Ollama, vLLM, llama.cpp, LM Studio, MLX.

El mensaje del mapa: "proveedor" no es sinónimo de "laboratorio". Consumir DeepSeek vía la API oficial china es distinto a consumirlo vía Fireworks o Together — mismo modelo, pricing diferente, política de datos diferente, compliance diferente. El arquitecto elige el **modelo** *y* el **canal**.

### 3.2 Top 9 empresas frontera (7 min)

La tabla central del bloque. Todas las versiones verificadas a 17-abr-2026:

| # | Empresa | Último flagship (API) | Modalidades | Razonador | Contexto | Precio in/out ($/M tok) | Open weights | Fortaleza |
|---|---|---|---|---|---|---|---|---|
| 1 | 🇺🇸 **Anthropic** | Claude Opus 4.7 *(16-abr-2026)* | Texto + visión HD 2576px | Sí (extended thinking) | 200K / 1M ent. | ~$5 / ~$25 | ❌ | Agentic coding, MCP nativo |
| 2 | 🇺🇸 **OpenAI** | GPT-5.4 *(5-mar-2026)* + Thinking / Pro / mini / nano | Texto + visión + audio Realtime + imagen | Sí (`reasoning_effort`) | 1M | ~$1.25 / ~$10 | ❌ | Ecosistema más ancho, voz |
| 3 | 🇺🇸 **Google** | Gemini 3.1 Pro *(abr-2026)* | Texto + visión + audio + **video nativo** | Sí (`thinking_level`) | 1M (2M Vertex) | ~$1.25 / ~$10 | ❌ | Long context real, video |
| 4 | 🇺🇸 **xAI** | Grok 4.20 v2 Reasoning + Grok 4.3 beta *(17-abr)* | Texto + visión | Sí | 256K | ~$3 / ~$15 | ❌ | Datos en vivo de X |
| 5 | 🇨🇳 **DeepSeek** | DeepSeek V3.2 + V3.2-Speciale | Texto | Sí (`deepseek-reasoner`) | 128K | ~$0.27 / ~$1.10 (off-peak -50%) | ✅ MIT | **Precio disruptivo**, MoE 671B |
| 6 | 🇨🇳 **Alibaba** | Qwen 3.6-Plus *(2-abr-2026)* | Texto + visión + UI-from-screenshot | Sí | 1M | ~$0.40 / ~$1.20 | ✅ (Medium, Apache 2.0) | Agentic coding, multilingüe |
| 7 | 🇨🇳 **Moonshot** | Kimi K2.6 Code Preview *(13-abr)* · base K2.5 MoE 1T/32B act. | Texto + visión | Sí | 1M | ~$1 / ~$3 | ✅ | Agent swarm (100 subagentes) |
| 8 | 🇨🇳 **Z.ai** (ex Zhipu) | **GLM-5.1** *(8-abr-2026)* MoE 754B | Texto + visión (GLM-5V) | Sí | — | ~$0.80 / ~$2.56 | ✅ MIT | **#1 SWE-Bench Pro**, 8h autónomas |
| 9 | 🇨🇳 **MiniMax** | **M2.7** *(abr-2026)* MoE 230B | Texto + visión + audio | Sí | 1M | ~$0.30 / ~$1.50 (hasta 50× más barato que frontier) | ✅ | Agentic self-improving, 100 tok/s |

Balance: **4 🇺🇸 + 5 🇨🇳 = 9**. Honorable mentions fuera del top 9: ByteDance Doubao, Cohere Command, Mistral Large (EU/GDPR).

Lectura del top 9:

- El mercado 2026 es verdaderamente multipolar. En LMArena y Artificial Analysis, los chinos empatan o superan a los gringos en varios ejes (especialmente SWE-Bench y coste).
- **Cinco de los nueve tienen pesos abiertos** (DeepSeek, Qwen parcial, Kimi, GLM, MiniMax). Los cuatro US son todos cerrados. Eso no es casualidad: refleja estrategias económicas distintas.
- La **diferencia de precio** entre el tier superior US y DeepSeek es ~15-50×. Para aplicaciones sensibles al coste, es una decisión existencial.
- La **multimodalidad** no está uniformemente distribuida: Gemini lidera en video; GPT-5.4 lidera en audio realtime; el resto se concentra en texto + visión.

**Sugerencia por vertical de negocio** — decisiones orientativas, no dogmas:

| Caso de negocio | Recomendación | Por qué |
|---|---|---|
| Coding agent / dev tool | Claude Opus · Qwen 3.6-Plus · GLM-5.1 · Kimi K2.6 | Líderes en SWE-Bench y agentic |
| Chatbot de alto volumen barato | DeepSeek V3.2 · GPT-5.4 nano · Gemini Flash | Precio por M token mínimo |
| App legal/sanitaria en EU | Mistral Large · Claude vía Bedrock EU | GDPR + ZDR |
| Video Q&A (transcripción, análisis) | Gemini 3.1 Pro | Único con video nativo |
| Voz en tiempo real | GPT-5.4 Realtime | API madura |
| Datos en vivo / social | Grok | Acceso X nativo |
| Self-host soberano | Qwen · DeepSeek · GLM · Kimi · MiniMax | Open-weights serios |

### 3.3 Open-weights: Llama vs los chinos (3 min)

Mención obligada. En 2023-2024 el rey open-weights era Llama. En 2026 **el liderazgo cambió de bando**. Meta sigue liberando pesos (Llama 4 Scout y Maverick están en HuggingFace) pero la frontera de calidad abierta ya vive en Qwen, DeepSeek, Kimi, GLM y MiniMax.

| Dimensión | **Meta Llama 4** | **Open-source chino** (Qwen / DeepSeek / Kimi / GLM / MiniMax) |
|---|---|---|
| Último | Llama 4 Scout (109B, 10M ctx) / Maverick (400B, 1M ctx) | Qwen 3.6-Plus, DeepSeek V3.2, Kimi K2.6, GLM-5.1, MiniMax M2.7 |
| Arquitectura | MoE 17B activos | MoE similar (32B-40B activos, 230B-754B totales) |
| Licencia | **Llama Community License** — restricciones a >700M MAU, atribución "Built with Llama" | **Apache 2.0 / MIT** — uso comercial libre, sin royalties |
| Calidad 2026 | Criticado en lanzamiento ("Meta panic button" — Interconnects). Perdió el liderazgo open. | GLM-5.1 #1 SWE-Bench Pro, DeepSeek V3.2-Speciale empata Gemini 3 Pro |
| Multilingüe | 12 idiomas declarados | Chino nativo + inglés fuertes; Qwen destaca en multilingüe |
| Ecosistema fine-tunes | El más amplio (años de ventaja en la comunidad) | Creciendo rápido; Qwen tiene decenas de variantes (Coder, Math, VL, Audio) |
| Infra de entrenamiento | NVIDIA H100 clusters | Mixto — GLM-5 se entrenó 100% en Huawei Ascend |
| Consideraciones políticas | Export controls potenciales; licencia US | Uso empresarial OK en Occidente; evitar API oficial china para PII |

Narrativa para la slide: *"El 'rey open-weights' cambió de bando. En 2023-2024 era Llama. En 2026 son Qwen, DeepSeek, GLM, Kimi y MiniMax. Llama sigue siendo relevante por ecosistema y licencia US-friendly, pero la frontera de calidad abierta ya vive en China."*

### 3.4 Chat vs API: no son la misma cosa (3 min)

Este punto es *el* error conceptual más frecuente en alumnos nuevos, y hay que desmontarlo explícitamente.

Cuando usas DeepSeek-chat (el producto web, `chat.deepseek.com`) puedes: generar imágenes, buscar en la web, subir PDFs, exportar gráficos, resolver tareas multi-paso con tools. Cuando usas la API de DeepSeek (`api.deepseek.com`) obtienes: un modelo de lenguaje que recibe texto y devuelve texto. Nada más.

La razón está en el marco de capas: **el chat del proveedor es ya una capa agéntica que ellos montaron sobre su propio LLM**. La API te da la capa 3 sola. La orquestación es tuya.

| | Chat (producto) | API |
|---|---|---|
| DeepSeek | imágenes, web search, PDFs, tool use | solo texto (V3.2) |
| ChatGPT | DALL-E, Sora, Code Interpreter, browsing | tú orquestas |
| Claude.ai | artifacts, projects, web search, MCP | Messages API + tú montas el resto |
| Gemini app | video gen, Deep Research | `generateContent` + tú |
| Kimi web | subagent swarm, visual coding | API + tú orquestas los 100 subagentes |

Esto tiene una implicación directa para el arquitecto: **lo que ves en el chat NO es lo que te da la API**. Si tu usuario final necesita que "el asistente busque en la web y cite fuentes", debes construir esa búsqueda tú (con Tavily, Exa, Perplexity Sonar, Brave, o un tool propio). El modelo solo razona sobre lo que le das.

Este es el momento para volver al diagrama de 3 capas: el chat web es capa 1+2+3 completas del proveedor; la API es capa 3 y tú construyes capas 1 y 2.

### 3.5 API vs self-host: cuándo cada uno (4 min)

Cuando consumes una API, los tamaños de modelo y el hardware no te importan: pagas por token y la infra es problema del proveedor. Cuando decides self-hostear, aparece una dimensión nueva: **los pesos ocupan RAM y requieren cómputo**.

**Cuándo SÍ tiene sentido self-hostear:**

1. **Compliance estricto**: datos clínicos, expedientes judiciales, secretos industriales, defensa.
2. **Coste a escala**: a partir de ~50-100M tokens/día una GPU dedicada amortiza contra la API. Umbral empírico: si tu factura mensual supera ~$30k, mide el self-host.
3. **Latencia ultra-baja controlada**: edge, on-device, robótica, copilots con <50 ms.
4. **Soberanía de datos**: sector público europeo, gobierno, secretos de estado.
5. **Fine-tuning profundo** sobre un modelo base con lógica de dominio.
6. **Disponibilidad offline**: barcos, operaciones remotas, contingencia.

**Cuándo NO tiene sentido (la mayoría de los casos):**

- Producto con <10M tokens/día → la factura API es de cientos de dólares; no justifica MLOps.
- Equipo sin experiencia GPU/Kubernetes → el TCO real (oncall, parcheo, observabilidad) te come.
- Necesitas frontier cerrado: no vas a reproducir GPT-5.4 ni Opus 4.7 en tu rack.
- Multimodal audio+video en tiempo real: infra muy cara.

**Tamaños y hardware — la tabla clave:**

| Tamaño | Memoria FP16 | Hardware mínimo | Hardware recomendado producción |
|---|---|---|---|
| 7B | ~14 GB | RTX 4070 / MacBook M2 16GB | 1× L4 / A10G |
| 13B | ~26 GB | RTX 4090 / MacBook M3 32GB | 1× A100 40GB |
| 40B | ~80 GB | 2× RTX 4090 | 1× A100 80GB / H100 |
| 70B | ~140 GB | 2× A100 80GB | 2× H100 / 1× H200 |
| 400B MoE (Llama 4 Maverick) | ~800 GB FP16 | 8× H100 NVL | cluster dedicado |
| 671B MoE (DeepSeek V3) | ~1.3 TB FP16 / ~380 GB FP8 | 8× H200 / 16× H100 | cuantización obligatoria |

**Cuantización** (Q4, Q5, Q8, FP8, AWQ) divide memoria 2-4× a cambio de 1-3 puntos de calidad. Para self-host casero es el default, no una optimización opcional.

**Dato práctico para la clase:** en un MacBook M-series de 16 GB RAM corren modelos 7-8B cuantizados con fluidez. Con 32 GB, modelos de 13B-30B. Con 64-128 GB (M3/M4 Max), 70B cuantizado. La era del *"necesitas un datacenter para hacer algo interesante"* terminó.

### 3.6 Dónde descargar modelos libres (4 min)

Si decides self-hostear, dos páginas dominan el ecosistema:

- **Hugging Face Hub** (https://huggingface.co) — el "GitHub del ML". ~1M modelos. Filtros por task, tamaño, licencia, formato. Aquí están los pesos originales en `safetensors` (FP16/BF16) y también las conversiones cuantizadas en formato GGUF hechas por la comunidad (bartowski, MaziyarPanahi son nombres frecuentes).
- **Ollama Library** (https://ollama.com/library) — ~300 modelos curados listos para correr con una línea. Es el "flujo fácil" para empezar.

Para un alumno que empieza, el flujo típico es este:

```bash
# Opción 1 — modelo de la librería oficial de Ollama
ollama pull qwen3-coder:30b
ollama run qwen3-coder:30b

# Opción 2 — cualquier GGUF de HuggingFace directamente
ollama run hf.co/bartowski/GLM-5.1-GGUF:Q4_K_M

# Opción 3 — servidor OpenAI-compatible local
ollama serve
# → escucha en http://localhost:11434/v1/chat/completions
# → tu código Python con base_url="http://localhost:11434/v1" simplemente funciona
```

Las herramientas principales, con cuándo usar cada una:

| Herramienta | Tipo | Para qué sirve | Hardware |
|---|---|---|---|
| **Ollama** | CLI + server HTTP | Dev local, servidor OpenAI-compat, integrar con código | CPU, GPU NVIDIA/AMD, Apple Silicon |
| **LM Studio** | GUI desktop | Descubrir, descargar y chatear con modelos; servidor OpenAI-compat | Windows, macOS (MLX), Linux |
| **llama.cpp** | CLI low-level | Edge, embebido, máximo control, cuantización custom | Cualquiera (CPU, Metal, CUDA, Vulkan, ROCm) |
| **vLLM** | Python server | Producción GPU, alto throughput (PagedAttention, continuous batching) | GPU NVIDIA/AMD |
| **MLX** | Framework Python | Apple Silicon optimizado, fine-tuning local | Mac M-series |
| **Jan** | GUI open-source | Alternativa 100% OSS a LM Studio | Cross-platform |
| **TGI** (HuggingFace) | Docker server | Alternativa a vLLM con integración HF Hub | GPU NVIDIA |

Formatos clave que el alumno debe conocer:

- **safetensors** — el formato PyTorch original de los pesos (full precision FP16/BF16).
- **GGUF** — cuantizado, para llama.cpp, Ollama y LM Studio. Sufijo `Q4_K_M`, `Q5_K_M`, `Q8_0` indica nivel de cuantización.
- **MLX** — formato Apple Silicon optimizado.
- **AWQ / GPTQ / FP8** — cuantizaciones para producción en GPU con vLLM o TensorRT-LLM.

**Punto didáctico que vale subrayar:** con Ollama + un laptop moderno, en 5 minutos el alumno puede estar corriendo **el mismo modelo de frontera open-weights** que la competencia china — Qwen 3.6 Medium, DeepSeek distill, Kimi K2 destilado — sin cuenta ni API key. Ese "wow" barato es parte del valor de esta clase.

### 3.7 Benchmarks para comparar (2 min)

Cierra el bloque. Tres recursos que vale la pena abrir en vivo:

- **LMArena** (https://lmarena.ai) — ranking por votos humanos ciegos. "Verdad de mercado" en conversación general.
- **Artificial Analysis** (https://artificialanalysis.ai) — calidad, precio, latencia, throughput actualizados semanalmente. La fuente más útil para decisiones de producto.
- **SWE-bench Verified** (https://www.swebench.com) — issues reales de GitHub con tests. El benchmark más relevante para coding agéntico.

Otros benchmarks que vale nombrar en una slide sin profundizar: **MMLU / MMLU-Pro** (conocimiento académico), **GPQA Diamond** (PhD-level), **AIME** (matemáticas), **ARC-AGI** (razonamiento novel), **MMMU** (multimodal), **LiveBench** (contaminación-resistente), **tau-bench** (agentes con tool use), **HLE — Humanity's Last Exam** (preguntas no-googleables).

Mensaje para el alumno: **no decidas por un solo benchmark**. Cruza capacidad (GPQA, SWE-bench), preferencia humana (LMArena), coste/latencia (Artificial Analysis), y **tu propio eval** sobre la tarea concreta de tu producto. Los benchmarks públicos se contaminan; lo que importa es tu métrica.

**Transición al Bloque 4:** *"Hemos visto quién ofrece los modelos y cómo elegirlos. Siguiente pregunta: ¿qué pegamento hay entre nuestro código y el modelo?"*

---

## Bloque 4 — Frameworks y protocolos · 12 min

### 4.1 Mapa de frameworks (3 min)

El ecosistema 2026 se ha estratificado. Ya no todos los frameworks compiten con todos. Cinco categorías:

- **Orquestación full-stack**: LangChain + LangGraph (grafos de agentes), LlamaIndex (RAG-first), Haystack (RAG enterprise EU), Semantic Kernel (Microsoft/Azure).
- **Librerías ligeras**: Pydantic AI (agentes tipados), Instructor (structured output), DSPy (programación declarativa de prompts), Mirascope.
- **Frontend / full-stack web**: Vercel AI SDK (TypeScript, estándar en Next.js).
- **Evaluación y observabilidad**: LangSmith, Langfuse (OSS), Phoenix (Arize), Braintrust, Helicone.
- **Agent runtimes oficiales por proveedor**: Claude Agent SDK, OpenAI Agents SDK, Google ADK.

**No hay un framework ganador absoluto.** El mercado eligió pluralidad. El trabajo del arquitecto es saber qué eje cubre cada uno y no mezclar mal.

Tabla de decisión rápida:

| Si necesitas... | Usa... |
|---|---|
| RAG pesado sobre documentos heterogéneos | LlamaIndex (+ LlamaParse) |
| Agente multi-step con ramas, HITL, checkpoints | LangGraph |
| Output estructurado validado | Pydantic AI · Instructor · `generateObject` de Vercel AI SDK |
| Stack Microsoft / Azure / C# | Semantic Kernel |
| App web con chat streaming y generative UI | Vercel AI SDK |
| Empezar simple y migrar cuando duela | SDK oficial del proveedor |
| Unificar múltiples proveedores | LiteLLM |
| Optimización automática de prompts | DSPy |

### 4.2 Framework vs SDK directo — el debate (3 min)

Pregunta frecuente: *"¿necesito LangChain?"*. Respuesta del curso: **empieza sin framework, añade cuando duela**.

A favor del framework:

- Velocidad de prototipado. 20 líneas para un agente RAG básico vs 300 con SDK puro.
- Integraciones gratis (loaders, vector stores, reranks). Nadie quiere escribir su conector número 47 a Confluence.
- Patrones resueltos (retries, streaming, tool calling, memoria con checkpoint).
- Observabilidad enchufable (LangSmith/Langfuse se conectan con una variable).
- Comunidad y contratación.

En contra:

- Abstracción leaky. Cuando falla, debugeas el framework *y* la API del proveedor.
- Upgrades rompen (LangChain 0.0 → 0.1 → 0.2 → 0.3 fue doloroso para muchos equipos).
- Esconde el prompt real. *"Show me the prompt"* — Hamel Husain.
- Tu caso usa el 5% del framework pero paga el 100% en dependencias, memoria y complejidad.
- Deuda técnica. Un junior que hereda el proyecto dos años después no entiende el flujo sin leer el framework entero.

**Heurística concreta del curso:**

1. **Semana 1 de cualquier proyecto LLM:** SDK oficial + Instructor/Pydantic para estructura + Langfuse para trazas. Suficiente para un MVP.
2. **Si necesitas RAG serio sobre muchos docs:** añade LlamaIndex (solo la parte de ingesta/retrieval).
3. **Si el flujo se vuelve un grafo con ramas, HITL y checkpoints:** LangGraph.
4. **Si es app web:** Vercel AI SDK desde el día 0. No reinventes streaming.
5. **Si llegas aquí sin adoptar nada:** probablemente no lo necesitabas.

### 4.3 MCP — el protocolo de tools (3 min)

Introducido por Anthropic en noviembre de 2024. Estandariza cómo los LLMs acceden a tools, resources y prompts externos.

- **Servidores MCP** exponen capacidades (tools, resources, prompts) vía JSON-RPC sobre stdio, HTTP o SSE.
- **Clientes MCP** (Claude Desktop, Cursor, Zed, VS Code, ChatGPT desktop) consumen esos servidores.
- **Adopción cross-vendor**: OpenAI lo adoptó en su Agents SDK (2025), Google en ADK, Microsoft en Semantic Kernel y Copilot Studio. Raro caso de estándar adoptado por todos los grandes.
- **Resuelve el problema M×N**: antes, M agentes × N tools requería M×N integraciones. Con MCP, M+N. Igual que LSP hizo con editores y lenguajes.

Servidores populares en 2026: filesystem, git, GitHub, GitLab, Slack, Postgres, SQLite, Google Drive, Linear, Notion, Jira, Sentry, Stripe, Cloudflare. Hay cientos de servidores comunitarios.

Por qué importa conceptualmente: **reduce el lock-in a cualquier framework concreto**. Si tu integración a Notion es un servidor MCP, da igual que el agente corra en LangGraph, Pydantic AI o SDK puro. El protocolo es el contrato.

### 4.4 ACP — el protocolo de agentes en IDEs (2 min)

Más nuevo, específico al ecosistema de desarrollo. Creado por Zed en 2025, adoptado por JetBrains (2025) y Cursor (marzo 2026).

Idea: **LSP para agentes**. Así como el Language Server Protocol permite que cualquier editor soporte cualquier lenguaje de programación a través de un estándar, ACP permite que cualquier agente se integre con cualquier IDE.

Ejemplo concreto: con ACP, puedes usar **Claude Code como agente dentro de JetBrains**, o **Cursor CLI como agente dentro de Zed**, sin integración ad-hoc. JetBrains lanzó un "ACP Agent Registry" en enero 2026 donde cualquiera puede publicar su agente.

Marca un patrón arquitectónico: **los IDEs de 2026 no son editores con IA, son orquestadores de agentes intercambiables**. El modelo mental clásico del IDE como "editor + plugins" queda obsoleto.

Para la clase: no hay que entrar al detalle del protocolo. Basta con que el alumno sepa que existe, que separa "agente" de "IDE", y que sigue el mismo patrón de M+N que MCP.

### 4.5 OAuth vs API key — cómo te conectas (1 min)

Tema subestimado pero muy práctico. Dos modelos de autenticación conviven:

- **API key**: el clásico. Pay-as-you-go por token. Variables `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`. Escalable, facturación por uso real.
- **OAuth / suscripción**: te autenticas con tu cuenta consumer. Límites por plan (Pro, Max), no por token. Ejemplos: **Claude Pro dentro de Claude Code**, **ChatGPT Plus dentro de Codex CLI**, **SuperGrok dentro de la CLI de xAI**.

Por qué importa: para un developer individual, suscribirse a Claude Pro y usar Claude Code con OAuth es a menudo **10× más barato** que consumir la API por uso — una factura plana de ~$20/mes vs $200+/mes de consumo frecuente. Para una empresa, API key es lo estándar porque permite atribuir coste por proyecto y negociar volumen.

Es una decisión con implicación económica real. En el curso vamos a trabajar con API keys porque es lo que se usa en producto para usuarios finales.

**Transición al Bloque 5:** *"Ya tenemos el modelo, el proveedor, el framework. Bajemos al nivel más concreto: ¿cómo se ve una llamada a un LLM en código?"*

---

## Bloque 5 — APIs de chat completions · 16 min

### 5.1 Anatomía de una llamada (4 min)

Desde marzo de 2023, `/v1/chat/completions` de OpenAI se convirtió en la forma canónica de hablar con un LLM. Conceptualmente es una función pura:

```
f(model, messages, params) → { choices, usage, finish_reason }
```

Los cuatro parámetros que importan al 90% de los casos:

- **`model`**: identificador del modelo (`claude-opus-4-7`, `gpt-5.4`, `deepseek-chat`, `gemini-3.1-pro`).
- **`messages`**: array con los turnos de la conversación. Cada turno tiene `role` (`system`, `user`, `assistant`, `tool`) y `content`.
- **`temperature`**: 0 a 2. Cuánta aleatoriedad introduce el muestreo. 0 = casi determinista (para extracción estructurada). 0.7 es típico para conversación.
- **`max_tokens`**: tope de tokens en la respuesta.

Otros parámetros que aparecen seguido: `top_p`, `stop`, `seed`, `response_format`, `tools`, `tool_choice`, `stream`.

Estructura de `messages` — cuatro roles:

```python
messages = [
    {"role": "system", "content": "Eres un analista electoral experto en Perú."},
    {"role": "user", "content": "¿Quién va ganando en Lima?"},
    {"role": "assistant", "content": "Según el último corte de ONPE..."},
    {"role": "user", "content": "¿Y la participación?"}
]
```

Respuesta tipo:

```json
{
  "choices": [
    { "message": {"role": "assistant", "content": "..."}, "finish_reason": "stop" }
  ],
  "usage": { "prompt_tokens": 523, "completion_tokens": 118, "total_tokens": 641 }
}
```

Ejemplo mínimo en Python:

```python
from openai import OpenAI
client = OpenAI()

resp = client.chat.completions.create(
    model="gpt-5.4",
    messages=[
        {"role": "system", "content": "Eres un asistente experto en elecciones peruanas."},
        {"role": "user", "content": "¿Quién va ganando en Lima?"}
    ],
    temperature=0.3,
)
print(resp.choices[0].message.content)
print(resp.usage.total_tokens)
```

Con esto — seis líneas — ya estamos hablando con un LLM frontier. Este es el punto más importante conceptualmente: **una llamada a un LLM es una llamada HTTP cualquiera**. No hay magia. Lo que ocurre antes del `create()` (diseño del prompt, preparación del contexto) y después (parseo, validación, tool loop) es donde vive el trabajo real de arquitectura.

### 5.2 El estándar OpenAI-compatible (3 min)

La razón del título *"el estándar de facto"* es concreta: **casi todo el ecosistema fuera de Anthropic y Google nativo habla el esquema de chat completions de OpenAI**.

Proveedores OpenAI-compatible relevantes en 2026:

| Proveedor | `base_url` |
|---|---|
| OpenAI | `https://api.openai.com/v1` |
| DeepSeek | `https://api.deepseek.com` |
| Groq | `https://api.groq.com/openai/v1` |
| Together AI | `https://api.together.xyz/v1` |
| Fireworks | `https://api.fireworks.ai/inference/v1` |
| Mistral | `https://api.mistral.ai/v1` |
| xAI | `https://api.x.ai/v1` |
| Gemini (modo compat) | `https://generativelanguage.googleapis.com/v1beta/openai` |
| Ollama local | `http://localhost:11434/v1` |
| vLLM local | `http://host:8000/v1` |
| LM Studio local | `http://localhost:1234/v1` |
| OpenRouter | `https://openrouter.ai/api/v1` |

**Cambiar de proveedor es cambiar dos líneas.** Esto no es una figura retórica: es literal.

Caveats honestos — lo que SÍ cambia entre proveedores:

- `response_format: json_schema` (structured outputs) solo funciona de verdad en OpenAI, Fireworks y algunos modelos de Together.
- Tool calling a veces acepta el campo `tools` pero el modelo alucina argumentos.
- `logprobs`, `seed`, `logit_bias` tienen soporte irregular.
- `prompt_caching` es específico de cada proveedor.

Regla: el *happy path* (`messages` + `temperature` + `stream`) es 100% portable. Las features avanzadas hay que probarlas.

### 5.3 Demo en vivo — cambiar de proveedor en 3 líneas (5 min)

El momento *wow* del bloque. Abrimos un notebook y proyectamos. Mismo código, varias ejecuciones:

```python
from openai import OpenAI
import os

# OpenAI (default)
client = OpenAI()

# DeepSeek — mismo SDK, dos variables cambiadas
client = OpenAI(
    api_key=os.environ["DEEPSEEK_API_KEY"],
    base_url="https://api.deepseek.com"
)

# Groq — también mismo SDK
client = OpenAI(
    api_key=os.environ["GROQ_API_KEY"],
    base_url="https://api.groq.com/openai/v1"
)

# Ollama local — no necesita cuenta ni red
client = OpenAI(
    api_key="ollama",
    base_url="http://localhost:11434/v1"
)

# El código de uso es IDÉNTICO para los cuatro:
resp = client.chat.completions.create(
    model="deepseek-chat",  # o "gpt-5.4", "llama-3.3-70b", etc.
    messages=[{"role": "user", "content": "Hola, en una frase: ¿qué es un LLM?"}],
)
print(resp.choices[0].message.content)
```

Mensaje del bloque: **la abstracción correcta para tu capa de modelo es una variable de entorno**. El proyecto `peru-elecciones` lo implementa exactamente así: cambiar de DeepSeek a Groq son dos variables en `.env`. Todo el código Go que consume el LLM queda intacto.

```go
// internal/llm/deepseek.go — peru-elecciones
ds := llm.NewDeepSeek(
    os.Getenv("DEEPSEEK_API_KEY"),
    os.Getenv("DEEPSEEK_BASE_URL"), // default https://api.deepseek.com
    os.Getenv("DEEPSEEK_MODEL"),
)
```

Para migrar de DeepSeek a Groq:
```bash
DEEPSEEK_BASE_URL=https://api.groq.com/openai/v1
DEEPSEEK_MODEL=llama-3.3-70b-versatile
```

No hay que tocar código. Esa es la promesa del estándar OpenAI-compatible.

### 5.4 Anthropic: la excepción notable (2 min)

Claude NO es OpenAI-compatible. Diferencias:

- Endpoint: `/v1/messages`, no `/v1/chat/completions`.
- System como parámetro separado, no como mensaje dentro de `messages`.
- `content` siempre array de bloques tipados (`text`, `image`, `tool_use`, `thinking`).
- `max_tokens` es obligatorio (en OpenAI es opcional).
- Header `anthropic-version` requerido.
- Roles permitidos: solo `user` y `assistant` (alternados). Tool results van como `role: user` con bloque `tool_result`.

Importante saberlo pero sin entrar al detalle. En producción se puede abstraer con **LiteLLM** (proxy que traduce 100+ APIs al schema OpenAI), o mantener dos ramas en tu cliente — una para OpenAI-compatible, otra para Anthropic.

Google Gemini también tiene su API nativa con `generateContent`, pero ofrece un **modo OpenAI-compatible** que es lo que casi todo el mundo usa en 2026.

### 5.5 Streaming con SSE (2 min)

Un LLM genera token a token. Esperar 8 segundos a que se forme una respuesta larga es UX inaceptable. **Server-Sent Events** permite empujar tokens al cliente a medida que se producen, reduciendo el tiempo-a-primer-token (TTFT) de ~5 s a ~200 ms percibidos.

En Python con el SDK:

```python
stream = client.chat.completions.create(
    model="gpt-5.4",
    messages=[{"role": "user", "content": "Cuenta hasta 5."}],
    stream=True,
)
for chunk in stream:
    if chunk.choices and (d := chunk.choices[0].delta.content):
        print(d, end="", flush=True)
```

Conceptualmente: el servidor abre una conexión HTTP con `Content-Type: text/event-stream`, envía chunks `data: {...}\n\n` y cierra al terminar. No es WebSocket, no es long-polling; es HTTP/1.1 estándar con un content type especial.

`peru-elecciones` tiene el endpoint `/api/stream` como stub. Migrarlo a SSE real es ejercicio de la clase 4.

### 5.6 Tool use — pincelada (0 min — mención final)

El patrón más importante del 2025-2026 después de chat completions. El ciclo:

1. Declaras tools como JSON Schema.
2. Mandas `messages` + `tools` al modelo.
3. El modelo decide: responde texto O pide llamar una tool.
4. **Tu código ejecuta la tool** (el modelo nunca ejecuta).
5. Devuelves el resultado al modelo.
6. El modelo continúa (puede pedir más tools o responder final).

En una línea: **el modelo genera intenciones estructuradas; el runtime es tuyo**.

Profundizamos en tool use y orquestación en la **clase 4**. Aquí basta con saber que existe, que vive en el mismo endpoint de chat completions (`tools=[...]`), y que es la base técnica de los agentes.

**Transición al Bloque 6:** *"Recapitulamos."*

---

## Bloque 6 — Cierre · 10 min

### 6.1 Tres take-aways (4 min)

Si el asistente solo recuerda tres cosas de esta clase, que sean estas.

**1. El LLM es una capa de infraestructura, no una feature.**

La diferenciación del producto vive alrededor — en la capa agéntica (loops, tools, memoria, MCP) y en el software que la administra (tu app). Todo el curso construye ese alrededor. Volvamos al diagrama de tres capas que abrimos en la apertura: lo que vas a aprender a construir en las próximas cuatro clases es la capa 2 y la capa 1, consumiendo la capa 3 como commodity.

**2. No hay "un mejor modelo".**

Hay un mejor modelo para *tu* combinación de tarea, coste, latencia y compliance. El mercado 2026 es multipolar: 4 frontier US, 5 frontier chinos, 5 open-weights competitivos, capas de inferencia especializada. La habilidad clave del arquitecto es **saber elegir y saber cambiar**. Para eso hay que estar permanentemente mirando LMArena, Artificial Analysis y las release notes de los proveedores.

**3. La API OpenAI-compatible es el TCP/IP de los LLMs.**

Diseña tu cliente como una variable de entorno. Cambiar de proveedor debería ser cambiar dos líneas, no refactorizar. `peru-elecciones` lo hace, y lo vamos a hacer en el resto del curso. Si un alumno sale de esta clase con el reflejo de abstraer su capa de modelo desde el día 0, cumplimos un objetivo importante.

### 6.2 Preview de la clase 2 — Prompt Engineering como sistema (2 min)

La clase 2 baja de la abstracción "modelo" a la abstracción "prompt". Qué vemos:

- Técnicas: zero-shot, few-shot, chain-of-thought, self-consistency.
- Diseño de system prompts.
- Output estructurado en JSON (`response_format`, Instructor, Pydantic).
- Validación y truncamiento.
- Práctica: construir un sistema multi-etapa de generación.

El enfoque: prompting no como arte, sino como **sistema** — versionado, testeado, iterado con datos.

### 6.3 Q&A (4 min)

Tiempo abierto para las preguntas inevitables:

- *"¿Y si uso LangChain?"* — respuesta corta: prueba sin él primero.
- *"¿Cómo elijo entre Claude y GPT?"* — respuesta: mide con tu eval, LMArena no es tu métrica.
- *"¿Merece la pena self-hostear?"* — respuesta: si tu factura API mensual no supera ~$30k, casi nunca.
- *"¿Qué pasa con la regulación europea?"* — respuesta: EU AI Act, GDPR, ZDR; hay que saber dónde está la línea de datos personales.
- *"¿Cuál es el stack que recomiendas para un MVP hoy?"* — SDK oficial + Instructor + Langfuse + Vercel AI SDK si es web.

---

## Anexo A — Checklist de verificación pre-clase

Los datos volátiles deben revisarse el mismo día de la clase:

- [ ] Último release Anthropic: https://docs.anthropic.com/en/release-notes/overview
- [ ] Último release OpenAI: https://platform.openai.com/docs/models
- [ ] Último release Gemini: https://ai.google.dev/gemini-api/docs/models
- [ ] Top del día en LMArena: https://lmarena.ai
- [ ] Pricing/latencia en Artificial Analysis: https://artificialanalysis.ai
- [ ] Release notes Z.ai/GLM: https://docs.z.ai/release-notes/new-released
- [ ] Release notes DeepSeek: https://api-docs.deepseek.com/updates/
- [ ] Release notes Qwen: https://qwenlm.github.io/
- [ ] Release notes Kimi: https://platform.moonshot.cn
- [ ] MiniMax: https://www.minimax.io
- [ ] Recordar tener Ollama con al menos un modelo chino instalado para demo del Bloque 3.6.

---

## Anexo B — Demos del día (lista de chequeo)

1. **Demo gancho peru-elecciones** (apertura, 3 min). Proyectar preguntas reales al asistente. Tener fallback grabado por si la conexión falla.
2. **Tokenización en tiktokenizer.vercel.app** (Bloque 2, 2-3 min). Frases ES / EN / ZH + código.
3. **Top 9 en LMArena y Artificial Analysis** (Bloque 3, 2 min). Proyectar ambos rankings en vivo.
4. **Ollama corriendo GLM o Qwen localmente** (Bloque 3.6, 2 min). Demostrar `ollama run` + llamada OpenAI-compatible desde Python.
5. **Cambio de `base_url` entre tres proveedores** (Bloque 5.3, 5 min). El momento *wow* de la clase.

---

## Anexo C — Fuentes principales

Agrupadas por tema. La bibliografía completa está en `referencias.md`.

**Papers fundacionales**
- Attention Is All You Need (Vaswani et al., 2017) — https://arxiv.org/abs/1706.03762
- GPT-3 (Brown et al., 2020) — https://arxiv.org/abs/2005.14165
- Foundation Models (Bommasani et al., 2021) — https://arxiv.org/abs/2108.07258
- Chain-of-Thought (Wei et al., 2022) — https://arxiv.org/abs/2201.11903
- InstructGPT (Ouyang et al., 2022) — https://arxiv.org/abs/2203.02155
- Constitutional AI (Bai et al., 2022) — https://arxiv.org/abs/2212.08073
- DPO (Rafailov et al., 2023) — https://arxiv.org/abs/2305.18290
- Lost in the Middle (Liu et al., 2023) — https://arxiv.org/abs/2307.03172
- DeepSeek-R1 (2025) — https://arxiv.org/abs/2501.12948

**Docs oficiales**
- Anthropic — https://docs.anthropic.com
- OpenAI — https://platform.openai.com/docs
- Google Gemini — https://ai.google.dev/gemini-api/docs
- DeepSeek — https://api-docs.deepseek.com
- Z.ai / GLM — https://docs.z.ai
- Qwen — https://qwenlm.github.io
- Kimi / Moonshot — https://platform.moonshot.cn
- MiniMax — https://platform.minimax.io
- xAI — https://docs.x.ai

**Protocolos**
- MCP — https://modelcontextprotocol.io
- ACP — https://zed.dev/acp

**Local / self-host**
- Ollama — https://ollama.com
- HuggingFace Hub — https://huggingface.co
- vLLM — https://docs.vllm.ai
- llama.cpp — https://github.com/ggml-org/llama.cpp

**Leaderboards y análisis**
- LMArena — https://lmarena.ai
- Artificial Analysis — https://artificialanalysis.ai
- SWE-bench — https://www.swebench.com
- HF Open LLM Leaderboard — https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard

**Opinión y debate**
- Simon Willison — https://simonwillison.net
- Chip Huyen — https://huyenchip.com/blog/
- Hamel Husain — https://hamel.dev
- Interconnects (Nathan Lambert) — https://www.interconnects.ai
- Latent Space — https://www.latent.space

---

**Fin del guion.** Total estimado en lectura pausada: ~25-30 minutos. Total en clase con demos y Q&A: **90 minutos**.

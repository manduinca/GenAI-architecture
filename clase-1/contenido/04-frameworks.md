# Bloque 4 — Frameworks del Ecosistema LLM: cuándo sí y cuándo no

> Dossier para la **Clase 1** del curso *Arquitectura de Aplicaciones con IA Generativa* (EscuelaIT, abril 2026). Nivel: desarrolladores intermedio-avanzados.
>
> **Nota metodológica:** los datos marcados con `[verificar]` (último release, stars, empresas) deben revisarse en GitHub/docs oficiales antes de la clase. URLs fuente en la sección final.

---

## 1. Mapa del ecosistema (abril 2026)

El ecosistema LLM en 2026 se ha estratificado en **cinco capas** bien diferenciadas, a diferencia del "cambrian explosion" de 2023-2024 donde todo competía con todo.

### 1.1. Orquestación y agentes (full-stack)

| Framework | Lenguaje | Foco | Estado 2026 |
|---|---|---|---|
| **LangChain + LangGraph** | Python / TS | Orquestación general + grafos de agentes | Consolidado; LangGraph es el producto estrella |
| **LlamaIndex** | Python / TS | Data framework, RAG-first | Consolidado en ingesta/retrieval |
| **Haystack 2.x** (deepset) | Python | Pipelines de búsqueda y RAG enterprise | Estable, nicho enterprise EU |
| **Semantic Kernel** | C# / Python / Java | Orquestación Microsoft/Azure | Plataforma oficial MS |
| **CrewAI** | Python | Agentes multi-rol ("tripulaciones") | Creciendo, comunidad fuerte |
| **AutoGen** (Microsoft Research) | Python / .NET | Conversación multi-agente | Refactorizado (v0.4+) en 2025 |
| **Google ADK (Agent Development Kit)** | Python / Java | Agentes para Vertex AI / Gemini | Lanzado 2025, tracción media |
| **Anthropic Claude Agent SDK + MCP** | TS / Python | "Anti-framework": tools primitivas | Momentum fuerte 2025-2026 |

### 1.2. Librerías ligeras y alternativas minimalistas

- **Pydantic AI** — agentes tipados con validación Pydantic nativa. Ha ganado tracción como "la opción pythónica limpia" en 2025.
- **Mirascope** — prompts como funciones Python tipadas; alternativa minimalista a LangChain.
- **Instructor** (Jason Liu) — *de facto* para structured output vía Pydantic antes de que los proveedores lo ofrecieran nativo. Sigue relevante en 2026 para multi-provider.
- **DSPy** (Stanford / Omar Khattab) — programación declarativa de prompts + optimización automática. Pasa de "curiosidad académica" a producción en 2025 con casos como JetBlue y Moody's.
- **Outlines** (dottxt-ai) — constrained generation con gramáticas/regex. El estándar para JSON garantizado en modelos open-source.
- **Guidance** (Microsoft) — menos momentum que Outlines; sigue vivo pero más nicho.

### 1.3. Frontend / full-stack

- **Vercel AI SDK** — estándar *de facto* en TypeScript para apps web (Next.js, SvelteKit, Nuxt). Versión 5.x en 2026 con generative UI, tool calling y streaming unificados.
- **Next.js RSC + AI SDK** — combinación canónica para apps conversacionales con server components.
- **LangChain.js** — existe pero con menor adopción que Vercel AI SDK; se usa cuando se requiere paridad con Python.
- **assistant-ui**, **CopilotKit** — componentes React para copilots embebidos.

### 1.4. Evaluación y observabilidad

- **LangSmith** (LangChain) — tracing + evals; líder por cuota dentro del ecosistema LangChain.
- **Langfuse** — open-source self-hostable; el favorito cuando se evita LangChain o cuando compliance exige on-prem.
- **Phoenix** (Arize) — OSS, fuerte en evals de RAG y OTel-native.
- **Braintrust** — comercial, fuerte en CI/CD de prompts.
- **Helicone** — proxy + observabilidad, low-friction.
- **Weave** (Weights & Biases) — integración con el stack W&B ML.

### 1.5. Agent runtimes emergentes y estándares

- **Claude Agent SDK** (Anthropic) — SDK oficial que empaqueta Claude Code como librería; fuerte en código, filesystem y subagentes.
- **OpenAI Agents SDK** — sucesor producción-ready de Swarm (experimental 2024 → deprecado → reemplazado por Agents SDK en 2025).
- **Anthropic Skills, Computer Use, Agent Skills** — capacidades primitivas del modelo, no frameworks.
- **MCP (Model Context Protocol)** — estándar abierto para conectar LLMs con tools y data. Adoptado en 2025 por OpenAI, Google, Microsoft, JetBrains, Zed, Replit — se considera *el TCP/IP de los agentes*.

---

## 2. Ficha por framework principal

### 2.1. LangChain + LangGraph

- **Lenguaje:** Python y TypeScript.
- **Bueno para:** prototipar rápido chains con integraciones exóticas (Milvus, Neo4j, etc.); LangGraph brilla en **agentes con estado, ramas condicionales, human-in-the-loop y checkpointing/resume**. LangGraph Platform ofrece despliegue gestionado.
- **Malo para:** aplicaciones simples "un prompt + un tool" donde introduce 3 capas de abstracción; debugging de errores internos; performance crítica (mucha indirección).
- **Curva:** media-alta. LCEL (LangChain Expression Language) y la dualidad LangChain/LangGraph siguen confundiendo.
- **Momentum:** **LangGraph > LangChain core** en 2026. LangChain core se percibe como "legacy-ish" para nueva arquitectura; LangGraph concentra la innovación. [verificar stars — referencia enero 2026: LangChain ~95k, LangGraph ~9k+ y creciendo rápido]
- **Último major:** LangChain 0.3.x estable en 2025; LangGraph 0.2/0.3 a lo largo de 2025-26 [verificar].
- **Empresas:** Klarna, Elastic, Replit, Rakuten, Uber (mencionadas en case studies oficiales).

### 2.2. LlamaIndex

- **Lenguaje:** Python (principal), TypeScript.
- **Bueno para:** RAG sobre documentos heterogéneos, **LlamaParse** para PDFs complejos con tablas/figuras, pipelines de ingesta, knowledge graphs sobre documentos, consultas multi-documento con agent routers. En 2026 se ha posicionado como **"data framework para LLMs"**, no como competidor full de LangChain.
- **Malo para:** orquestación compleja de agentes (LangGraph es superior); frontend; workflows no-RAG.
- **Curva:** media. La API de `Workflows` (introducida en 2024) simplificó mucho.
- **Momentum:** estable, ~40k stars [verificar]. LlamaParse (SaaS) es su motor de monetización.
- **Último major:** LlamaIndex v0.12+ con Workflows y AgentWorkflow [verificar].
- **Empresas:** KPMG, Uber (docs internos), Notion (referencia), múltiples casos enterprise vía Databricks.

### 2.3. Semantic Kernel

- **Lenguaje:** C# (first-class), Python, Java.
- **Bueno para:** stacks Microsoft/Azure, integración con M365/Copilot/Azure OpenAI; enterprise con compliance; equipos .NET que no quieren salir del ecosistema. Soporte first-class para MCP y Agent Framework (unificación con AutoGen anunciada en 2025).
- **Malo para:** startups Python-first; casos donde se requieren últimas features de investigación (SK prioriza estabilidad).
- **Curva:** media para devs C#; plugins + planners tienen ergonomía Microsoft clásica.
- **Momentum:** estable dentro del ecosistema MS; fuera de él, limitado. [verificar stars ~25k]
- **Último major:** Semantic Kernel 1.x + integración con Microsoft Agent Framework (lanzado 2025).
- **Empresas:** Microsoft (internamente), bancos europeos, sector público, integradores MS.

### 2.4. Haystack (deepset)

- **Lenguaje:** Python.
- **Bueno para:** **RAG enterprise con compliance EU**, pipelines componibles, búsqueda híbrida madura, despliegue on-prem. Haystack 2.x (reescritura 2024) es más limpio que la 1.x.
- **Malo para:** agentes complejos; casos donde se prefiere Python moderno tipado (la DSL es YAML/Python mixto).
- **Curva:** media.
- **Momentum:** estable, nicho. [verificar ~18k stars]
- **Último major:** Haystack 2.x serie.
- **Empresas:** Airbus, Deutsche Telekom, Siemens — clientes mencionados por deepset.

### 2.5. Pydantic AI

- **Lenguaje:** Python.
- **Bueno para:** **agentes tipados end-to-end**; teams que ya usan Pydantic en FastAPI; validación automática de inputs/outputs; type-safe tool calling; dependency injection limpio. Creado por el equipo de Pydantic/Logfire en 2024, ha explotado en 2025-26 como la alternativa "FastAPI-like" a LangChain.
- **Malo para:** grafos de agentes muy complejos (mejor LangGraph); TypeScript (no existe port oficial).
- **Curva:** baja si ya conoces Pydantic.
- **Momentum:** **uno de los frameworks con mayor crecimiento 2025-26**. Integración nativa con Logfire (observabilidad del mismo equipo).
- **Último major:** 0.x estable, rumbo a 1.0 [verificar].
- **Empresas:** adopción mayormente en startups Python modernas; el equipo Pydantic reporta uso interno en GitHub, Anthropic y otros.

### 2.6. Vercel AI SDK

- **Lenguaje:** TypeScript.
- **Bueno para:** **apps web con chat streaming, generative UI, tool calling** en Next.js/React/Svelte/Vue. Abstrae proveedores (OpenAI, Anthropic, Google, xAI, Mistral, locales). Primitivas como `streamText`, `generateObject`, `useChat` son el estándar en el frontend.
- **Malo para:** agentes multi-step server-side complejos (su foco es UX); pipelines de evals; casos no-web.
- **Curva:** baja para devs React/Next.
- **Momentum:** **dominante en TypeScript**; v5 (2025) unificó tool calling y structured output, agregó primitivas de agentes (`Agent`, `experimental_*`).
- **Último major:** AI SDK 5.x [verificar versión exacta].
- **Empresas:** Vercel, Perplexity (partes de UI), Cursor (UI), miles de startups Next.js.

### 2.7. DSPy

- **Lenguaje:** Python.
- **Bueno para:** **optimización automática de prompts y pipelines** mediante "teleprompters" (MIPRO, BootstrapFewShot); programación declarativa donde defines *signatures* (input→output) en lugar de prompts string; investigación y pipelines complejos donde la búsqueda de mejores prompts vale la pena.
- **Malo para:** proyectos simples; equipos que necesitan control fino de los prompts exactos; iteración rápida UI-driven.
- **Curva:** **alta** — cambio mental significativo ("no escribas prompts, describe contratos").
- **Momentum:** salió del mundo académico en 2025; Databricks lo incorporó; Moody's, JetBlue publican casos.
- **Último major:** DSPy 2.5+ / DSPy 3 [verificar].
- **Empresas:** Databricks, Moody's, JetBlue, Stanford.

---

## 3. El debate "framework vs SDK directo"

### 3.1. A favor del framework

- **Velocidad de prototipado.** 20 líneas para un agente RAG básico vs 300 con SDK puro.
- **Integraciones gratis.** Loaders para 100+ fuentes, vector stores, embeddings, reranks. Nadie quiere escribir su 47º conector a Confluence.
- **Patrones ya resueltos.** Retries, streaming, tool calling, memoria de conversación con checkpoint en Postgres.
- **Observabilidad enchufable.** LangSmith/Langfuse se conectan con una variable de entorno.
- **Comunidad y contratación.** Buscar "LangChain developer" da candidatos; buscar "nuestro framework interno" no.

### 3.2. En contra

- **Abstracción leaky.** Cuando falla, debes entender la capa del framework *más* la API del proveedor. El stack trace atraviesa 6 archivos del framework.
- **Overhead de rendimiento** y dependencias pesadas (decenas de paquetes transitivos).
- **Deuda técnica.** Upgrades rompen (LangChain v0.1 → 0.2 → 0.3 fue doloroso para muchos equipos).
- **Sobre-generalización.** El framework optimiza para "cualquier caso", no el tuyo.
- **Context limit del equipo.** Cada framework nuevo es una API más que aprender, revisar y versionar.

### 3.3. La escuela minimalista

- **Hamel Husain** ("Your AI Product Needs Evals", *Fuck you, show me the prompt*): ejecuta el SDK directo, loguea los prompts literales, y construye herramientas específicas para tu caso antes que adoptar un framework abstracto. Filosofía: *"Don't build frameworks, build tools."*
- **Simon Willison** empuja la misma línea: su librería `llm` es un CLI sobre los SDKs, no un framework. Su mantra: "Empieza con `requests` y un string".
- **Eugene Yan**, **Jason Liu** (Instructor) y **Shreya Shankar** en el mismo eje: prioriza evals y datos sobre frameworks.
- **Swyx / Latent Space** ha documentado la ola "post-LangChain" con entrevistas recurrentes a equipos que migraron fuera.

### 3.4. Recomendación para el curso

**Regla del curso:** *Empieza con el SDK oficial del proveedor. Añade un framework solo cuando el dolor lo justifique.*

Heurística concreta:
1. **Semana 1 de cualquier proyecto LLM:** SDK directo (Anthropic, OpenAI) + Instructor/Pydantic para estructura + Langfuse para trazas.
2. **Si necesitas RAG serio sobre muchos documentos:** incorpora LlamaIndex (solo la parte de ingesta/retrieval).
3. **Si el flujo se vuelve un grafo con ramas, reintentos condicionales y HITL:** LangGraph.
4. **Si es app web:** Vercel AI SDK desde el minuto 0 (no tiene sentido reinventar streaming).
5. **Si llegas aquí sin haber adoptado nada:** probablemente no lo necesitabas.

---

## 4. MCP (Model Context Protocol)

### 4.1. Qué es

Protocolo abierto introducido por **Anthropic en noviembre 2024** para estandarizar cómo los LLMs acceden a tools, resources y prompts externos. Define:

- **Servidores MCP:** exponen capacidades (tools, resources, prompts) vía JSON-RPC sobre stdio/HTTP/SSE.
- **Clientes MCP:** aplicaciones host (Claude Desktop, Cursor, Zed, VS Code, ChatGPT desktop) que consumen esos servidores.
- **Transporte:** stdio local, HTTP streamable, SSE.

### 4.2. Por qué gana tracción

- **Adopción cross-vendor.** OpenAI anunció soporte MCP en Agents SDK (2025), Google en ADK, Microsoft en Semantic Kernel y Copilot Studio — el raro caso de un estándar adoptado por todos los grandes proveedores.
- **Desacopla tools de agentes.** Un servidor MCP (ej: GitHub, Postgres, Slack) funciona con Claude, GPT, Gemini y agentes custom **sin reescribir integraciones**.
- **Resuelve el problema del "M×N":** antes, M agentes × N tools requería M×N integraciones; con MCP, M+N.
- **Ecosistema creciente:** cientos de servidores comunitarios publicados en 2025.

### 4.3. Servidores MCP populares (2025-26)

- **Oficiales Anthropic:** filesystem, git, GitHub, GitLab, Slack, Postgres, SQLite, Google Drive, Puppeteer/playwright, fetch, memory.
- **Comunitarios:** Linear, Notion, Jira, Sentry, Stripe, Cloudflare, Figma, Obsidian, Brave Search, Exa, Perplexity.
- **Enterprise:** Databricks, Snowflake, Atlassian (oficiales) han publicado MCPs en 2025.

### 4.4. Cómo cambia el panorama

- Reduce el *lock-in* a frameworks — si tu integración a Notion es un servidor MCP, da igual que uses LangGraph, Pydantic AI o el SDK puro.
- Empuja a los frameworks a ser **capas finas sobre MCP** en vez de reimplementar integraciones.
- El debate "framework vs SDK" se suaviza: el framework aporta orquestación, pero las tools vienen de MCP.

---

## 5. Evolución del ecosistema 2023-2026

- **2023 — LangChain boom.** Todos escribían en LangChain. Superó 50k stars en meses. Explosión de librerías derivadas. `llama-index` (antes `gpt_index`) gana tracción en RAG.
- **2024 — Backlash y diversificación.** Posts virales ("LangChain is a headache", "Why we left LangChain"). Refactors v0.1/v0.2. Nacen alternativas: Pydantic AI, Mirascope, CrewAI. Instructor y DSPy ganan maturity. OpenAI suelta Swarm (experimental). Anthropic presenta **MCP (nov 2024)** y **Computer Use**.
- **2025 — Consolidación y estándares.** MCP se adopta cross-vendor. LangGraph eclipsa a LangChain core. Vercel AI SDK v5. OpenAI reemplaza Swarm con Agents SDK productivo. Microsoft unifica AutoGen + Semantic Kernel en el **Microsoft Agent Framework**. Anthropic lanza **Claude Agent SDK** y **Skills**. Google lanza **ADK**.
- **2026 — Madurez.** El mercado se divide claramente: (a) *runtimes oficiales* por proveedor (Claude Agent SDK, OpenAI Agents SDK, ADK), (b) *frameworks neutrales maduros* (LangGraph, LlamaIndex, Pydantic AI), (c) *MCP como capa común de tools*, (d) *observabilidad* como categoría separada (Langfuse, LangSmith, Phoenix, Braintrust). El debate ya no es "qué framework", sino "cuánto framework".

---

## 6. Decisión práctica en tabla

| Si necesitas... | Usa... | Por qué |
|---|---|---|
| RAG pesado sobre documentos heterogéneos | **LlamaIndex** (+ LlamaParse para PDFs) | Ingest + retrieval es lo mejor de la industria |
| Agente multi-step con ramas, HITL y checkpoints | **LangGraph** | State machine explícito, checkpointer, resume |
| Output estructurado validado | **Pydantic AI** (Python) / **Instructor** (multi-provider) / `generateObject` (TS) | Type-safe, validación automática |
| Stack Microsoft / Azure / C# | **Semantic Kernel** + Microsoft Agent Framework | Integración first-class con Azure y M365 |
| App web con chat streaming y generative UI | **Vercel AI SDK** (+ Next.js) | Estándar TS, `useChat`, streaming out-of-the-box |
| Empezar simple y migrar cuando duela | **SDK oficial del proveedor** (Anthropic / OpenAI) + Instructor | Menos magia, más control, menos deuda |
| Unificar múltiples proveedores LLM | **LiteLLM** (proxy) o abstracción del AI SDK | 100+ proveedores detrás de una API tipo OpenAI |
| Optimización automática de prompts | **DSPy** | Teleprompters optimizan pipelines |
| JSON garantizado con open-source models | **Outlines** | Constrained decoding real |
| Conectar tools heterogéneas cross-vendor | **MCP** | Estándar industria, un servidor = todos los agentes |
| RAG enterprise on-prem EU | **Haystack 2.x** | Madurez y soberanía de datos |
| Agentes multi-rol colaborativos (marketing, research) | **CrewAI** | Metáfora de "tripulación" productiva |
| Observabilidad OSS self-hosted | **Langfuse** o **Phoenix** | Sin vendor lock-in, OpenTelemetry-native |

---

## 7. Señales de alerta: cuándo el framework no vale la pena

Cuando detectes dos o más de estas señales, **considera bajar de capa de abstracción**:

1. **Pasas más tiempo leyendo el source del framework que el de tu aplicación.** El stack trace del bug tiene más frames del framework que tuyos.
2. **Copias/pegas ejemplos sin entenderlos porque "así funciona".** La API cambió 3 veces en 6 meses (*cough* LangChain 0.0 → 0.1 → 0.2).
3. **Un cambio trivial** (añadir un campo al output) requiere entender tres abstracciones nuevas (Runnable, Output Parser, Callback).
4. **Tu equipo debate más sobre "la forma framework" que sobre el problema del usuario.**
5. **El framework esconde los prompts reales.** No puedes ver exactamente qué se envió al LLM sin activar flags de debug obscuros — *"fuck you, show me the prompt"* (Hamel).
6. **Los evals son difíciles de montar** porque las entradas y salidas están enterradas en objetos del framework.
7. **Deploy y observabilidad pelean con el framework** (ej: el streaming del framework no se lleva bien con tu edge runtime).
8. **Tu caso usa el 5% del framework** — paga el 100% en dependencias, memoria y complejidad.
9. **Un junior del equipo no entiende el flujo.** Si el flujo es `prompt → LLM → parse → tool → LLM → output` y tu código tiene 8 niveles de envoltorios, sobra.
10. **Los upgrades del framework dan miedo.** Si `pip install -U` da ansiedad, ya es deuda técnica.

**Regla de oro:** *si el framework te ayuda los primeros 20%, pero te estorba el último 80%, has elegido el nivel de abstracción equivocado.*

---

## 8. Fuentes

### Repos y documentación oficial

- LangChain — https://github.com/langchain-ai/langchain / https://python.langchain.com
- LangGraph — https://github.com/langchain-ai/langgraph / https://langchain-ai.github.io/langgraph/
- LlamaIndex — https://github.com/run-llama/llama_index / https://docs.llamaindex.ai
- Haystack — https://github.com/deepset-ai/haystack / https://haystack.deepset.ai
- Semantic Kernel — https://github.com/microsoft/semantic-kernel / https://learn.microsoft.com/en-us/semantic-kernel/
- Microsoft Agent Framework — https://github.com/microsoft/agent-framework
- AutoGen — https://github.com/microsoft/autogen / https://microsoft.github.io/autogen/
- CrewAI — https://github.com/crewAIInc/crewAI / https://docs.crewai.com
- Google ADK — https://github.com/google/adk-python / https://google.github.io/adk-docs/
- Pydantic AI — https://github.com/pydantic/pydantic-ai / https://ai.pydantic.dev
- Mirascope — https://github.com/Mirascope/mirascope
- Instructor — https://github.com/jxnl/instructor / https://python.useinstructor.com
- DSPy — https://github.com/stanfordnlp/dspy / https://dspy.ai
- Outlines — https://github.com/dottxt-ai/outlines
- Guidance — https://github.com/guidance-ai/guidance
- Vercel AI SDK — https://github.com/vercel/ai / https://ai-sdk.dev
- LiteLLM — https://github.com/BerriAI/litellm

### Agent runtimes y estándares

- Claude Agent SDK — https://github.com/anthropics/claude-agent-sdk-python / https://docs.claude.com/en/api/agent-sdk/overview
- OpenAI Agents SDK — https://github.com/openai/openai-agents-python / https://openai.github.io/openai-agents-python/
- OpenAI Swarm (legacy) — https://github.com/openai/swarm
- Model Context Protocol — https://modelcontextprotocol.io / https://github.com/modelcontextprotocol
- MCP servers oficiales — https://github.com/modelcontextprotocol/servers
- Anthropic Skills / Computer Use — https://docs.claude.com

### Observabilidad y evals

- LangSmith — https://smith.langchain.com / https://docs.smith.langchain.com
- Langfuse — https://github.com/langfuse/langfuse / https://langfuse.com
- Phoenix (Arize) — https://github.com/Arize-ai/phoenix / https://docs.arize.com/phoenix
- Braintrust — https://www.braintrust.dev
- Helicone — https://github.com/Helicone/helicone / https://helicone.ai
- Weave (W&B) — https://github.com/wandb/weave / https://wandb.ai/site/weave

### Escuela minimalista y crítica

- Hamel Husain — https://hamel.dev (ver "Your AI Product Needs Evals", "Fuck you, show me the prompt")
- Simon Willison — https://simonwillison.net / https://llm.datasette.io
- Eugene Yan — https://eugeneyan.com (patrones LLM, "Task-Specific LLM Evals")
- Jason Liu (Instructor) — https://jxnl.co
- Shreya Shankar — https://www.sh-reya.com
- Latent Space (swyx) — https://www.latent.space

### Post-mortems e historia del ecosistema

- "Why we no longer use LangChain" (Octomind blog, 2024) — https://www.octomind.dev/blog/why-we-no-longer-use-langchain-for-our-ai-agents
- Hacker News debates (LangChain backlash 2024) — busca "LangChain" en https://news.ycombinator.com
- State of AI Engineering reports (Latent Space, anuales)

---

## Resumen para la clase

1. **No hay framework ganador absoluto.** El mercado se estratificó: runtimes oficiales por proveedor, frameworks neutrales maduros (LangGraph, LlamaIndex, Pydantic AI), y MCP como capa de tools.
2. **Empieza sin framework.** Añade cuando el dolor lo justifique. El SDK directo + Instructor + Langfuse te lleva muy lejos.
3. **MCP ha cambiado las reglas.** Reduce el lock-in a cualquier framework concreto.
4. **Elige por eje, no por marca.** RAG → LlamaIndex. Grafo de agentes → LangGraph. Tipado → Pydantic AI. Web UX → Vercel AI SDK. MS stack → Semantic Kernel.
5. **Señal de alerta #1:** si peleas con el framework más que con el problema, bajaste demasiado la capa de abstracción — o la subiste.

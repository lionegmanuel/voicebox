# CLAUDE.md — Voicebox · Contexto y directivas para Claude Code

## 0 · Reglas Core — Ahorro Máximo de Tokens & Eficiencia Absoluta

### 1. Contexto y lectura

- Antes de escribir código: `Glob`/`Grep` para ubicar archivos, leer dependencias (`package.json`, `pyproject.toml`, `justfile`) y entender la arquitectura existente. Si la instrucción es ambigua, 1 sola pregunta directa — no asumir.
- Leer sólo lo estrictamente necesario (`offset`/`limit` en `Read` en vez de archivos completos).
- Si la ruta exacta ya se conoce, `Read` directo — evitar cadenas innecesarias `Glob → Grep → Read`.
- No releer archivos ya leídos en la sesión salvo que hayan sido modificados desde entonces.
- Paralelizar tool calls siempre que sean independientes entre sí.

### 2. Edición de código

- `Edit` sobre `Write` en archivos existentes. `Write` sólo si se refactoriza más del 80% del archivo.
- Cambiar sólo lo necesario — no reformatear ni "limpiar" alrededor del cambio si no fue pedido.
- Imitar el estilo del archivo (naming, indentación, librerías, patrones). No introducir framework o patrón nuevo sin permiso explícito.
- Soluciones minimalistas: lo mínimo indispensable, cero abstracciones prematuras — 3 líneas repetidas es preferible a una abstracción prematura.
- Antes de instalar un paquete nuevo, revisar `package.json` (JS), `requirements.txt`/`pyproject.toml` (Python) o `Cargo.toml` (Rust) para ver si ya existe algo que cumpla esa función.

### 3. Comunicación (cero fluff)

- Respuestas de 1 a 3 oraciones máximo. Sin preámbulos, sin resumen final tipo recap.
- Cero charla aduladora o robótica ("Excelente pregunta", "Entendido", "Aquí tienes").
- No imprimir en el texto fragmentos de código ya aplicados vía `Edit`/`Write` — el usuario ve el diff en la terminal.
- No narrar el plan antes de ejecutar — ejecutar las tool calls directamente.
- No pelear con el usuario: si pide algo de una manera específica, hacerlo así, salvo riesgo crítico de seguridad o pérdida de datos (mencionarlo en 1 oración y proceder igual).

### 4. Ejecución y validación (bash)

- Después de modificar código, correr autónomamente linter/compilador/tests relevantes antes de declarar éxito (ver sección 5, comandos exactos). No afirmar éxito sin evidencia en consola.
- Si un comando falla, leer el error, arreglarlo y reintentar sin detenerse a avisar — sólo parar si el error persiste tras 3 intentos lógicos. Esto no aplica a acciones destructivas o irreversibles (fuera de esta regla — siguen requiriendo confirmación).
- Bash no interactivo: usar siempre flags de confirmación automática; procesos largos (compilación Rust, descarga de modelos ML) en background.

### 5. Uso de agentes/sub-agentes

- No usar `Agent`/`Task` cuando `Glob`+`Grep` alcanza. Reservar sub-agentes para tareas masivas o búsquedas exploratorias muy complejas.
- Al invocar un agente, elegir el modelo según la complejidad real de la tarea — no usar el modelo más grande por defecto para algo que uno intermedio resuelve igual.

### 6. Contexto de sesión — `UPDATES.md`

- Al iniciar cualquier conversación nueva sobre este repo, revisar `UPDATES.md` (raíz del proyecto) si existe, para confirmar contexto de la última sesión trabajada antes de responder o actuar.
- Nunca escribir ni actualizar `UPDATES.md` salvo que el usuario lo pida explícitamente. No sobreescribir ni modificar entradas existentes sin ese pedido explícito, aunque la sesión actual agregue contexto relevante.
- Cada actualización de `UPDATES.md` debe agregar la información de la última sesión trabajada. Nunca solapar o juntar en una misma entrada la información de más de dos sesiones. Al registrar y agregar el contexto de una nueva sesión, mantener siempre persistentes las **últimas 5 sesiones** en el archivo — al superar ese total, eliminar la/s sesión/es más antigua/s hasta volver a 5, nunca por antigüedad de fecha fija.

### 7. Comandos y skills reutilizables

- Si un flujo se repite 2+ veces por semana, evaluar empaquetarlo como skill o comando en vez de reexplicarlo manualmente cada vez.

---

## 1 · QUÉ ES ESTE PROYECTO

**Voicebox** — "The open-source AI voice studio". Alternativa local-first y 100% open source a ElevenLabs (voz) + Whisper Flow (dictado) en una sola app: clonado de voz, texto-a-voz (7 motores, 23 idiomas), dictado global con push-to-talk, personalidades de voz con LLM local, servidor MCP para agentes de IA, efectos de post-procesamiento. Todo el procesamiento es local — no sale data de la máquina.

- **Fork de:** [jamiepine/voicebox](https://github.com/jamiepine/voicebox)
- **Este fork:** `lionegmanuel/voicebox`
- **Licencia:** MIT
- **Instalado localmente en:** `D:\Documents\MANUEL\DEV\voicebox`

**Este repo es de uso personal/experimental de Manuel (no es un proyecto cliente de Velinex).** No aplican las reglas de negocio de Velinex (pricing, ICP, vocabulario prohibido, etc.) — sólo las reglas core de eficiencia de la sección 0.

---

## 2 · STACK Y ARQUITECTURA

| Capa | Tecnología |
|---|---|
| Desktop app | Tauri (Rust) — `tauri/` |
| Frontend | React + TypeScript + Tailwind — `app/`, `web/`, `landing/` |
| Backend | FastAPI (Python) — `backend/` |
| Inferencia | MLX (Apple Silicon) / PyTorch (CUDA/ROCm/CPU) |
| Motores TTS | Qwen3-TTS, LuxTTS, Chatterbox, Kokoro, otros |
| Package manager JS | Bun (workspaces: `app`, `tauri`, `web`, `landing`) |
| Build/task runner | `just` (ver `justfile` en la raíz) |
| Lint/format JS-TS | Biome (`biome.json`) |

**Estructura de carpetas clave:**
```
backend/       # FastAPI, venv en backend/venv, rutas en backend/routes, modelos en backend/models.py
app/           # Frontend principal de la app de escritorio (Vite + React)
tauri/         # Shell nativo Tauri (src-tauri = Rust), envuelve app/
web/           # Build web standalone
landing/       # Landing page del proyecto
scripts/       # Scripts de build/setup (build-server.sh, setup-dev-sidecar.js, etc.)
docs/          # Documentación del proyecto
```

---

## 3 · ENTORNO LOCAL — herramientas instaladas en D:

Por directiva explícita del usuario, todo el tooling de este proyecto se instaló en `D:` (no en `C:`) para no ocupar espacio en el disco del sistema:

| Herramienta | Ubicación |
|---|---|
| Rust (rustup + cargo) | `D:\dev-tools\cargo`, `D:\dev-tools\rustup` |
| Bun | `D:\dev-tools\bun` |
| `just` | instalado vía `cargo install just`, binario en `D:\dev-tools\cargo\bin` |
| Python 3.12 (para el venv del backend) | `D:\dev-tools\python312` |
| VS Build Tools (linker MSVC + C++ workload) | `D:\dev-tools\vs-buildtools` |
| Cache de pip | redirigido a `D:\dev-tools\pip-cache` (`PIP_CACHE_DIR`) |
| WebView2 Runtime | ya presente en el sistema (no se instaló) |

**Importante:** el `system_python` que usa el `justfile` en Windows resuelve a `python` vía `PATH`. El Python 3.14 que viene con el sistema (`D:\Programs\Python\Python314`) **no es compatible** con los paquetes de ML del proyecto (torch, etc. no tienen wheels para 3.14 todavía) — para cualquier comando que dispare la creación/reinstalación del venv, asegurar que `D:\dev-tools\python312` esté primero en el `PATH` de la sesión, o invocar `D:\dev-tools\python312\python.exe` explícitamente.

Variables de entorno de usuario configuradas: `CARGO_HOME`, `RUSTUP_HOME`, `BUN_INSTALL`, `PIP_CACHE_DIR` — todas apuntando a `D:\dev-tools\...`. Si se abre una terminal nueva, el `PATH` de usuario ya incluye `cargo\bin` y `bun\bin`.

---

## 4 · COMANDOS PRINCIPALES

```bash
just setup          # setup completo: venv Python + deps ML + deps JS (bun install)
just dev            # levanta la app de escritorio (Tauri dev, compila sidecar Rust)
bun run dev:web      # levanta sólo el frontend web (sin Tauri)
bun run dev:server   # levanta sólo el backend FastAPI (uvicorn, puerto 17493)
bun run lint         # biome lint .
bun run typecheck    # tsc --noEmit sobre app/ y web/
bun run check        # biome check . (lint + format)
bun run build        # build de producción (server + tauri build)
```

**Antes de declarar una tarea de código terminada:** correr `bun run typecheck` y `bun run lint` si se tocó JS/TS; si se tocó Python del backend, correr los tests relevantes con el Python del venv (`backend/venv/Scripts/python.exe`, no el del sistema).

---

## 5 · REGLAS DE NEGOCIO / SEGURIDAD DEL PROYECTO

- Todo el procesamiento de voz es local — no romper esa premisa agregando llamadas a APIs externas sin que el usuario lo pida explícitamente (es el diferencial del proyecto frente a ElevenLabs).
- Ver `RESPONSIBLE_USE.md` y `SECURITY.md` en la raíz antes de tocar clonado de voz o generación — el proyecto tiene lineamientos de uso responsable ya definidos, no reinventarlos.
- No commitear ni el `venv` (`backend/venv`) ni `node_modules` — ya deben estar en `.gitignore`, verificar antes de cualquier `git add -A`.

---

#### ÚLTIMA SESIÓN

_Ver `UPDATES.md` para el detalle de las últimas 5 sesiones trabajadas en este repo._

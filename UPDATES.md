# UPDATES.md — Voicebox · Historial de sesiones

> Registrar sólo cuando el usuario lo pida explícitamente. Mantener siempre las últimas 5 sesiones — al superar ese total, eliminar la(s) más antigua(s) hasta volver a 5. Nunca solapar más de una sesión por entrada.

---

### Sesión 2026-07-23 — Instalación inicial del proyecto

Manuel encontró el repo [jamiepine/voicebox](https://github.com/jamiepine/voicebox) (alternativa open-source a ElevenLabs para clonado de voz y TTS local) y le hizo fork a `lionegmanuel/voicebox`. Pidió instalar todo el proyecto en su PC, con la restricción explícita de que ningún tooling nuevo se instale en el disco `C:` — todo a `D:` en la medida de lo posible.

**Trabajo realizado:**

1. Clonado del fork en `D:\Documents\MANUEL\DEV\voicebox`.
2. Instalación de todo el tooling con datos redirigidos a `D:\dev-tools\`:
   - Rust (rustup + cargo) vía `rustup-init.exe -y --default-toolchain stable --profile default`, con `CARGO_HOME`/`RUSTUP_HOME` apuntando a `D:\dev-tools\cargo` / `D:\dev-tools\rustup`.
   - Bun vía script oficial (`bun.sh/install.ps1`), con `BUN_INSTALL=D:\dev-tools\bun`.
   - Visual Studio Build Tools 2022 (workload C++/VCTools) vía `winget`, instalado con `--location D:\dev-tools\vs-buildtools` — necesario porque Rust MSVC requiere el linker `link.exe`, que no venía con el toolchain de rustup solo.
   - Python 3.12 vía `winget` (`Python.Python.3.12`), instalado en `D:\dev-tools\python312` — necesario porque el Python 3.14 que ya tenía el sistema no tiene wheels compatibles con los paquetes de ML del proyecto (torch, etc.).
   - `just` (task runner del proyecto) vía `cargo install just --root D:\dev-tools\cargo`.
3. `just setup` corrido con éxito: creó el venv de Python en `backend/venv` (usando el Python 3.12 de D:, no el del sistema) e instaló todas las dependencias de ML + `bun install` de los workspaces JS (799 paquetes).
4. Detectado que el cache de pip se llenó en `C:\Users\MANUEL\AppData\Local\pip\cache` (4.7GB) porque no estaba redirigido de entrada — se limpió ese cache y se configuró `PIP_CACHE_DIR=D:\dev-tools\pip-cache` como variable de entorno de usuario para que futuras instalaciones de pip no vuelvan a escribir en C:.
5. Verificado que WebView2 Runtime ya estaba presente en el sistema (no hubo que instalarlo).
6. Creados `CLAUDE.md` y este `UPDATES.md` en la raíz del repo, siguiendo la misma estructura y reglas core de ahorro de tokens/eficiencia que se usan en el resto de los proyectos de Manuel.

**Estado al cierre de la sesión:** proyecto listo para levantarse con `just dev` (o `bun run dev:web` / `bun run dev:server` para levantar partes sueltas). Todavía no se corrió `just dev` end-to-end para confirmar que la app de escritorio abre correctamente (la primera corrida va a compilar el sidecar de Rust/Tauri, puede tardar varios minutos).

**Pendiente / a verificar en la próxima sesión:**
- Confirmar que `just dev` levanta la app de escritorio sin errores.
- Si se reinicia la PC o se abre una terminal nueva, confirmar que `cargo`, `bun` y `just` resuelven bien desde el `PATH` de usuario (ya persistido, no debería requerir pasos manuales).

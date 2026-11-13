# SolidCraft — README de esta copia local

> Documento de **esta copia de trabajo**, en español. El [`README.md`](README.md) del proyecto
> (en inglés) sigue intacto y es la referencia oficial. Este archivo no forma parte del upstream:
> describe qué se instaló aquí, qué se cambió y **qué falta por verificar**.

## Índice

1. [Resumen rápido](#1-resumen-rápido)
2. [Créditos y origen](#2-créditos-y-origen)
3. [Qué es SolidCraft](#3-qué-es-solidcraft)
4. [Cambios y mejoras de esta copia](#4-cambios-y-mejoras-de-esta-copia)
   - [4.1 Botón IA en la barra superior](#41-botón-ia-en-la-barra-superior)
   - [4.2 Ventana «Conectar asistentes de IA»](#42-ventana-conectar-asistentes-de-ia)
   - [4.3 Instrucciones por cliente en el motor](#43-instrucciones-por-cliente-en-el-motor)
   - [4.4 Cadenas de idioma nuevas](#44-cadenas-de-idioma-nuevas)
   - [4.5 Documentación](#45-documentación)
   - [4.6 Prueba de regresión](#46-prueba-de-regresión)
   - [4.7 Archivos tocados](#47-archivos-tocados)
   - [4.8 Lo que no se tocó](#48-lo-que-no-se-tocó)
5. [Instalación en este equipo (fuera del repositorio)](#5-instalación-en-este-equipo-fuera-del-repositorio)
6. [Cómo se usa](#6-cómo-se-usa)
7. [Verificación: qué está comprobado y qué no](#7-verificación-qué-está-comprobado-y-qué-no)
8. [Continuar el trabajo](#8-continuar-el-trabajo)
9. [Licencias y agradecimientos](#9-licencias-y-agradecimientos)

---

## 1. Resumen rápido

| | |
|---|---|
| **Proyecto** | SolidCraft — CAD 3D paramétrico open source en Rust puro |
| **Creador original** | **Brennen Herbruck** ([`@bherbruck`](https://github.com/bherbruck)) |
| **Mantenimiento hoy** | [Tangency Labs](https://github.com/tangency-labs) + contributors |
| **Esta copia** | commit `7b90a309a` del repositorio oficial, con cambios locales |
| **Lo añadido aquí** | Botón **IA** en la barra superior + ventana con instrucciones exactas para conectar Claude, Codex, OpenCode, Cursor y 6 asistentes más por MCP |
| **Estado** | Código escrito, **no compilado** (esta máquina no tiene Rust ni linker MSVC) |
| **Licencia** | Código MIT OR Apache-2.0 · marca e interfaz bajo `LICENSE-BRAND` |

---

## 2. Créditos y origen

SolidCraft **no** es un proyecto de esta carpeta: es software open source ajeno, clonado desde su
repositorio oficial. Todo el mérito del programa es de sus autores.

| | |
|---|---|
| **Creador original** | **Brennen Herbruck** — [`@bherbruck`](https://github.com/bherbruck) · `brennen.herbruck@gmail.com` |
| Primer commit | `dafce1f4e`, 7 de octubre de 2026 — *«M0: workspace, geometry foundation, sketch constraint solver, kernel boundary»* |
| **Mantenimiento actual** | [**Tangency Labs**](https://github.com/tangency-labs) y los contributors de SolidCraft |
| Repositorio | <https://github.com/tangency-labs/solidcraft> (origen: `github.com/bherbruck/solidcraft`) |
| Web / documentación | <https://solidcraft.io> · <https://docs.solidcraft.io/> |
| Comunidad | [Discord](https://discord.gg/FtU4NQNVDH) |
| Licencia del código | **MIT OR Apache-2.0**, a elección |
| Licencia de marca e interfaz | [`LICENSE-BRAND`](LICENSE-BRAND): UI kit, look (iconos, glifos, temas), tokens de diseño y el nombre/logo |
| Kernel B-rep | [`solidcore`](https://github.com/tangency-labs/solidcore) (Apache-2.0), vendorizado en `solidcore/` |
| Base de esta copia | commit `7b90a309a` — *«Viewport: the grid's spacing and reach live in solidcraft-viewport…»* (13 nov 2026) |

Créditos completos de terceros en [`ATTRIBUTION.md`](ATTRIBUTION.md),
[`THIRD-PARTY-LICENSES.txt`](THIRD-PARTY-LICENSES.txt) y [`NOTICE`](NOTICE). SolidCraft es
independiente de Autodesk, Dassault, PTC, Siemens y del resto de fabricantes CAD.

---

## 3. Qué es SolidCraft

CAD 3D paramétrico escrito en **Rust puro**, sin kernel de terceros:

- **Sketch** con solver de restricciones propio y timeline de features con reconstrucción incremental.
- **Kernel B-rep** propio (`solidcore`, vendorizado) para operaciones exactas.
- **Renderizado GPU** con `wgpu`, y UI propia sobre `egui` (escritorio y navegador).
- **Cuatro formas de uso**: app de escritorio, app web (wasm), CLI headless (`solidcraft-cli`) y
  **servidor MCP**, para que un agente de IA modele con los mismos comandos que la interfaz.

Escala de esta copia (medida hoy sobre el árbol de trabajo):

| Métrica | Valor |
|---|---|
| Líneas Rust en `crates/` | 1 079 637 (≈1,08 M) |
| Líneas Rust en `solidcore/` | 1 417 895 (≈1,42 M) |
| Tests (`#[test]`) | 22 871 (15 136 en `crates/` + 7 735 en `solidcore/`) |

---

## 4. Cambios y mejoras de esta copia

Todo lo de esta sección es **trabajo nuevo sobre el código original**: 4 archivos modificados
(+203 / −22 líneas) y 1 archivo nuevo.

### 4.1 Botón IA en la barra superior

- **Dónde:** barra de aplicación (`app_bar`), a la izquierda, junto a Guardar / Deshacer / Rehacer.
- **Icono:** `agent_link`, ya existente en `crates/look` — **no se añadieron assets nuevos**.
- **Tooltip:** «Conectar asistentes de IA…» (id `ui.connect_ai`).
- **Archivo:** `crates/ui-egui/src/toolbar.rs`
  - nueva acción `connect_ai` en el vector `actions` de la barra;
  - se atiende en `BarAction::Button` llamando a `crate::help::open_connect(app)`.
- **Comportamiento:** abre la ventana **sin escribir nada** hasta que se pulsa *Conectar*. Respeta
  el bloqueo de ventanas del propio programa (`help::available`), así que no se abre encima de otro
  diálogo.

**Antes:** para conectar un asistente había que usar el menú `?` (que solo volcaba una lista de
resultados) o escribir `solidcraft-cli mcp setup` en una terminal.

### 4.2 Ventana «Conectar asistentes de IA»

Reescrita en `crates/ui-egui/src/help.rs`. Contenido, de arriba abajo:

1. **Explicación** de qué es MCP y qué hace el botón (sin jerga: «los mismos comandos que ejecuta la
   barra de herramientas»).
2. **Botón primario** «Conectar los asistentes de este equipo»: ejecuta el mismo `agent_setup` que
   `solidcraft-cli mcp setup`. Cada archivo se respalda como `.bak` y la escritura es atómica; las
   entradas propias del usuario no se tocan.
3. **Resultado** por cliente: configurado, sin cambios, actualizado o error.
4. **Sección «Añadir el servidor a mano»**, una fila plegable por asistente con:
   - ruta exacta del archivo de configuración,
   - fragmento listo para pegar (JSON o TOML según el cliente),
   - botón **Copiar** individual,
   - aviso cuando SolidCraft no encontró ese cliente en el equipo.
5. **Botón «Copiar todo»** con las instrucciones de todos los asistentes.
6. Estado nuevo `HelpState::connect_open` y entrada pública `help::open_connect(app)`.
7. El ítem del menú `?` ahora usa el icono `agent_link` y abre **esta misma ventana** conectando al
   instante (se conserva el comportamiento anterior).

**Los 10 clientes cubiertos** (los mismos que soporta `solidcraft-cli mcp setup`), cada uno con la
forma real que su programa espera:

| Cliente | Archivo de configuración | Forma de la entrada |
|---|---|---|
| OpenCode | `~/.config/opencode/opencode.json` | `mcp` → `command` en array, `type: local` |
| Claude Desktop | `%APPDATA%\Claude\claude_desktop_config.json` (y la copia MSIX) | `mcpServers` |
| Claude Code | `~/.claude.json` | `mcpServers` + `type: stdio` |
| Codex | `~/.codex/config.toml` | `[mcp_servers.solidcraft]` en TOML |
| Cursor | `~/.cursor/mcp.json` | `mcpServers` |
| Gemini CLI | `~/.gemini/settings.json` | `mcpServers` |
| GitHub Copilot CLI | `~/.copilot/mcp-config.json` | `type: local`, `tools: ["*"]` |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` | `mcpServers` |
| Zed | `%APPDATA%\Zed\settings.json` | `context_servers` |
| VS Code | `%APPDATA%\Code\User\mcp.json` | `servers` + `type: stdio` |

### 4.3 Instrucciones por cliente en el motor

`crates/engine/src/agent_setup.rs`:

- **Nuevo** `pub struct Instructions { client, label, file, snippet, found }`.
- **Nuevo** `pub fn instructions(env, exe) -> Vec<Instructions>`: devuelve **todos** los clientes
  soportados con su archivo y su fragmento, **aunque no estén instalados** (antes solo se podía
  enumerar los detectados).
- **Nuevo** `pub fn client_label(client) -> &'static str`: nombre legible de cada cliente.
- **Nuevo** `fn snippet(shape, exe, args) -> String`: genera el fragmento real — JSON con la clave de
  lista correcta por cliente (`mcp`, `mcpServers`, `context_servers`, `servers`) o TOML para Codex.
- **Refactor** `detect` → `detect_any(env, only, existing_only)`: un solo recorrido sirve para «qué
  está instalado» y «qué se le puede explicar». `detect` conserva el comportamiento previo
  (`existing_only = true`).
- Claude Code aparece también en las instrucciones aunque no exista `~/.claude.json` (antes la
  comprobación de existencia lo ocultaba).

### 4.4 Cadenas de idioma nuevas

`assets/locales/pending/connect-ai-button.json` (**nuevo**), siguiendo la regla del repo: una PR
añade **un archivo** en `pending/` y nunca edita los catálogos generados.

| id | Inglés | Español |
|---|---|---|
| `ui.connect_ai` | Connect AI assistants… | Conectar asistentes de IA… |
| `connect.run` | Connect the assistants on this computer | Conectar los asistentes de este equipo |
| `connect.manual` | Add the server by hand | Añadir el servidor a mano |
| `connect.copy` | Copy | Copiar |
| `connect.copy_all` | Copy all | Copiar todo |
| `connect.intro` | *qué es MCP y qué hace Conectar* | *qué es MCP y qué hace Conectar* |
| `connect.restart` | *reiniciar el asistente; respaldos `.bak`* | *reiniciar el asistente; respaldos `.bak`* |
| `connect.not_found` | *cliente no encontrado en este equipo* | *cliente no encontrado en este equipo* |

Los demás idiomas caen a inglés hasta que se traduzcan (comportamiento documentado del proyecto).

### 4.5 Documentación

`docs/mcp.md`: la vía rápida de configuración menciona ahora el **botón IA** de la barra de
aplicación además de *Help > Connect AI assistants…*, y describe qué hace la ventana.

### 4.6 Prueba de regresión

En el módulo `tests` ya existente de `crates/engine/src/agent_setup.rs` (el repo no permite archivos
de test por bug):

`instructions_name_the_file_and_the_entry_for_a_client_setup_skips` comprueba que, con un `home`
vacío (ningún cliente instalado):

- se listan todos los clientes igualmente y ninguno figura como `found`;
- OpenCode apunta a `opencode.json` y su fragmento es `mcp` → `command: [exe, "mcp"]`,
  `type: "local"`;
- Codex genera **TOML** (`[mcp_servers.solidcraft]`), no JSON;
- Cursor, Zed y VS Code usan su propia clave de lista (`mcpServers`, `context_servers`, `servers`).

### 4.7 Archivos tocados

| Archivo | Estado | Qué cambia |
|---|---|---|
| `crates/engine/src/agent_setup.rs` | modificado | `Instructions`, `instructions()`, `client_label()`, `snippet()`, `detect_any()` + test |
| `crates/ui-egui/src/help.rs` | modificado | `connect_open`, ventana nueva con instrucciones, `open_connect()`, guardas |
| `crates/ui-egui/src/toolbar.rs` | modificado | botón `connect_ai` y su manejo en `BarAction::Button` |
| `docs/mcp.md` | modificado | el botón IA como vía rápida |
| `assets/locales/pending/connect-ai-button.json` | **nuevo** | 8 cadenas en inglés y español |
| `README.local.md` | **nuevo** | este documento |

### 4.8 Lo que no se tocó

- `README.md`, `AGENTS.md`, `CONTRIBUTING.md` y el resto de documentación original.
- Iconos, temas, fuentes y cualquier asset: se reutiliza el icono existente `agent_link`.
- El kernel, el solver, el formato de diseño y el protocolo MCP.
- Los catálogos de idioma generados (`assets/locales/<locale>.json`).

---

## 5. Instalación en este equipo (fuera del repositorio)

Estos cambios afectan a la instalación, no al código, y por eso **no aparecen en el diff**:

- **SolidCraft 0.41.0 portable**, instalado sin permisos de administrador en
  `%LOCALAPPDATA%\Programs\SolidCraft\` (`solidcraft.exe` + `solidcraft-cli.exe`).
  ZIP oficial `solidcraft-0.41.0-windows-x64-portable.zip` con **SHA256 verificado**:
  `0fa9db32d31ed4109c5782a460b8ce63cf25df8a443961529685993996d89d04`.
- **Servidor MCP conectado a OpenCode**: `%USERPROFILE%\.config\opencode\opencode.json` con la
  entrada `solidcraft`, añadida por `solidcraft-cli mcp setup --client opencode`. Expone 22
  herramientas (`execute`, `batch`, `measure`, `screenshot`, `body_topology`, `skill_reference`…).
- **Modo en vivo** (opcional): abrir la app y activar
  **Preferences → Assistant → Agents through MCP → “Let AI agents connect”**. Solo entonces el
  servidor MCP se engancha a la ventana abierta; si no, trabaja en memoria (headless).

---

## 6. Cómo se usa

1. Abrir SolidCraft.
2. Pulsar el botón **IA** de la barra superior (o `?` → *Connect AI assistants…*).
3. Pulsar **Conectar los asistentes de este equipo**.
4. Reiniciar el asistente (OpenCode, Claude, Codex…).
5. Pedirle una pieza, por ejemplo:

   > Crea una placa de 80 × 60 × 5 mm con cuatro agujeros pasantes de 4,5 mm a 10 mm de los bordes y
   > redondea las esquinas verticales con radio 6.

Si un asistente no aparece, su fila en *«Añadir el servidor a mano»* trae la ruta y el fragmento
exactos para pegarlo.

---

## 7. Verificación: qué está comprobado y qué no

> **Aviso honesto:** esta máquina **no tiene toolchain de Rust ni linker de MSVC**, así que el cambio
> **no se ha compilado**. El código está escrito contra las APIs que existen en el repo (leídas una a
> una), pero falta la verificación del compilador y las capturas de pantalla.

| Comprobación | Estado |
|---|---|
| APIs usadas (`ui_kit::Window`, `Text`, `Button`, `section`, `publish_target`, `agent_setup`) | ✅ leídas en el código actual |
| Sintaxis y tipos Rust (`cargo check`) | ❌ **no ejecutado** |
| Test nuevo de `instructions()` | ✅ escrito · ❌ no ejecutado |
| JSON de cadenas de idioma | ✅ validado con `ConvertFrom-Json` |
| `cargo xtask ci` (capas, licencias, assets, i18n, clippy, tests) | ❌ no ejecutado |
| Captura del botón y de la ventana | ❌ pendiente |

Nota técnica: la ventana usa `ui_kit::Window` con `actions("", "Close", None)`, de modo que solo
aparece el botón **Cerrar**; los botones propios (Conectar, Copiar) van dentro del cuerpo, con sus
rectángulos publicados para los tests de escenario (`connect:run`, `connect:copy:<cliente>`).

---

## 8. Continuar el trabajo

### Requisitos (según `rust-toolchain.toml`)

- Rust **1.99.1** con `rustfmt`, `clippy` y el target `wasm32-unknown-unknown`.
- En Windows, *Visual Studio Build Tools* (MSVC) para enlazar.

### Comandos

```sh
rustup toolchain install 1.99.1 --component rustfmt clippy --target wasm32-unknown-unknown
cargo xtask ci                 # capas, licencias, assets, i18n, clippy, tests
cargo run --release -p solidcraft
```

### Checklist antes de cualquier PR

- [ ] `cargo xtask ci` en verde.
- [ ] Captura del botón IA y de la ventana, con OpenCode, Claude y Codex a la vista.
- [ ] `cargo xtask i18n` y `cargo xtask lists` (ids nuevos en `pending/`).
- [ ] `cargo xtask skills` si cambian los skills citados en `AGENTS.md`.
- [ ] PR en borrador, un tema por PR, título y primera frase comprensibles sin contexto
      (`skills/solidcraft-contributing/SKILL.md`).

### Reglas del proyecto que hay que respetar

- **Clean-room**: nada de código, iconos ni textos de productos CAD comerciales.
- Sin código GPL/LGPL/AGPL, ni siquiera leerlo (`cargo xtask licences --check`).
- Nada de `unwrap` / `expect` / `panic` / `unsafe` fuera de tests: la entrada es hostil.
- Las decisiones de modelo usan geometría exacta, no mallas.
- Todo es un comando; las llamadas programáticas nunca abren diálogos.
- Documentación, código y mensajes de commit **en inglés** para el upstream.

---

## 9. Licencias y agradecimientos

- Código: **MIT OR Apache-2.0**, a elección del usuario.
- Marca e interfaz: [`LICENSE-BRAND`](LICENSE-BRAND) — el UI kit, el look (iconos, glifos, temas) y
  los tokens de diseño. Se pueden usar y modificar **dentro de SolidCraft**; usarlos en otro
  producto requiere permiso escrito de los mantenedores.
- Fuentes, dependencias y assets de terceros: [`ATTRIBUTION.md`](ATTRIBUTION.md),
  [`THIRD-PARTY-LICENSES.txt`](THIRD-PARTY-LICENSES.txt) y [`NOTICE`](NOTICE).

Gracias a **Brennen Herbruck** por crear SolidCraft y a **Tangency Labs** y a todos los contributors
por mantenerlo abierto. Este documento solo describe el trabajo local hecho **sobre** su programa;
el programa sigue siendo suyo.
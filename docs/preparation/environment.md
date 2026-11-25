# Verificación del entorno · P-005 y P-005.1

## P-005.1 · Estado vigente

**Preparación operativa básica completada el 25 de noviembre de 2026.** El bloqueo registrado en P-005 quedó resuelto. La terminal convencional selecciona Node.js 24.22.3 LTS desde la instalación del sistema. Docker responde a las consultas. Quedan pendientes la ejecución efectiva de contenedores y las pruebas de aplicación.

| Comprobación | Terminal convencional · Briant + Fernando | Ejecución de Codex desde el repositorio |
| --- | --- | --- |
| `node --version` | `v24.22.3` | `v24.22.3` |
| `npm.cmd --version` | `11.5.1` | `11.5.1` |
| `openspec.cmd --version` | `2.2.0` | `2.2.0` |
| `openspec.cmd list` | `No active changes found.` | `No active changes found.` |
| `where.exe node` | Resultado no informado. | Primera coincidencia: `C:\Program Files\nodejs\node.exe`. El runtime incluido en Codex aparece después. |
| `docker info` | Resultado no informado. | Docker Desktop respondió con servidor 28.0.4, tres contenedores registrados y cero en ejecución. |
| `docker ps` | Resultado no informado. | Listado vacío, sin contenedores activos. |

Los siete comandos repetidos por Codex terminaron con código 0. El registro de la solicitud y la respuesta está en la [bitácora P-005.1](../bitacora/P-005.1.md).

## P-005 · Antecedente histórico

La tabla y los comandos siguientes corresponden al estado observado durante P-005, antes de la actualización de Node.js.

Comprobaciones realizadas el 25 de noviembre de 2026. **Confirmado** indica un resultado observado. **Pendiente** identifica una acción requerida. **Bloqueado** indica que el requisito no se cumple en el entorno comprobado. **No comprobado** identifica pruebas fuera de esta ejecución.

| Elemento | Estado | Resultado |
| --- | --- | --- |
| Sistema operativo | Confirmado | Windows 11 Home Single Language, 64 bits, compilación 26200. |
| Node.js seleccionado para el proyecto | Confirmado | **24.22.3 LTS Krypton**, reproducible mediante `.nvmrc`. Node.js publica la versión y distribuye binarios para Windows. |
| Node.js seleccionado por el PATH | Bloqueado | La terminal prioriza el ejecutable del sistema **22.14.0**. No cumple la referencia del proyecto ni el mínimo declarado por OpenSpec 2.2.0. |
| Node.js dentro de Codex | Confirmado | **24.22.3** obtenido mediante un ejecutable incluido en Codex. En la sesión también aparece como candidato secundario de resolución. Se usó explícitamente para verificar OpenSpec sin cambiar el PATH. |
| Node.js 24.22.3 en una terminal convencional | Pendiente | No se comprobó una instalación de 24.22.3 accesible fuera de Codex. Activar esa versión y volver a verificar la resolución del ejecutable y OpenSpec. |
| npm | Confirmado | **11.5.1** en el entorno actual. Volver a comprobar al seleccionar Node.js 24.22.3 en una terminal convencional. |
| Git | Confirmado | **2.49.0.windows.1**. |
| Docker CLI | Confirmado | **28.0.4**, build `b8034c0`. |
| Docker Compose | Confirmado | **2.34.0-desktop.1**. |
| Motor Docker | Confirmado | El servidor respondió **28.0.4** y `docker ps` terminó correctamente sin contenedores activos. El bloqueo de conexión registrado en P-004 no se reprodujo. |
| Ejecución de contenedores | No comprobado | La ejecución se limitó a consultas del motor. |
| OpenSpec | Confirmado | **2.2.0**. El manifiesto instalado declara Node.js `>=22.20.0`. El arranque y las consultas funcionaron también bajo el runtime de Node.js 24.22.3 incluido en Codex. |
| Integración de OpenSpec | Confirmado | Perfil `custom`, esquema `spec-driven`, alcance de proyecto y 11 skills en `.agents/skills/`. `bulk-archive` y `onboard` quedan deshabilitados. |
| Aplicación, Compose del proyecto y CI | No comprobado | Los artefactos ejecutables se incorporarán en sus incrementos. |

La identificación de LTS y la disponibilidad del binario se comprobaron en el [anuncio oficial de Node.js 24.22.3](https://nodejs.org/en/blog/release/v24.22.3) y el [archivo de distribución](https://nodejs.org/en/download/archive/v24.22.3). Se seleccionó esa revisión exacta para que `.nvmrc` sea reproducible y satisfaga el mínimo del CLI.

## Comandos y resultados observados

| Consulta | Resultado |
| --- | --- |
| `Get-CimInstance Win32_OperatingSystem` | Identificó Windows, arquitectura y compilación. |
| `node --version`, `Get-Command node -All`, `where.exe node` | El ejecutable convencional 22.14.0 aparece primero. La ejecución explícita de Node.js incluido en Codex devolvió 24.22.3. |
| `npm.cmd --version`, `npm.cmd root -g`, `git --version` | Confirmaron npm, la ubicación necesaria para consultar el manifiesto de OpenSpec y la versión de Git. |
| `docker --version`, `docker compose version`, `docker info --format '{{.ServerVersion}}'`, `docker ps` | Las versiones coincidieron con la tabla. Las dos consultas al motor devolvieron código 0. |
| `openspec.cmd --version`, `openspec.cmd list` | Informaron 2.2.0 y ningún change activo. El éxito de estas consultas con el entorno actual no acredita que Node.js 22.14.0 cumpla el requisito del paquete. |
| `openspec config get profile`, `openspec config get workflows` | Perfil `custom`. Workflows: `propose`, `explore`, `new`, `continue`, `apply`, `update`, `ff`, `sync`, `archive` y `verify`. |
| `openspec list --json`, `openspec list --specs --json`, `openspec list --archived --json` | Con Node.js 24.22.3 de Codex se identificó la raíz correcta, con cero changes activos, cero archivados y cero specs. |
| `openspec validate --all --no-interactive --json`, `openspec schema validate spec-driven` | El CLI informó cero elementos de aplicación y esquema válido. Aún no existe contenido de un change que pueda validarse. |

## Acciones que quedaron pendientes en P-005

Activa Node.js **24.22.3** en una terminal convencional y vuelve a ejecutar `node --version`, `npm --version`, `openspec --version` y `openspec list`. Antes de las tareas que usen contenedores vuelve a comprobar el motor con `docker info` y `docker ps`. Esta comprobación no descargó ni ejecutó imágenes de prueba.

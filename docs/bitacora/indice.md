# Índice de la bitácora de desarrollo

Cada fila distingue el estado consignado en la entrada de la evidencia disponible en los artefactos. Las entradas históricas se conservan como fueron encontradas. Usa la [plantilla](plantilla.md) para nuevos registros conforme a [EA-006](../requirements.md#ea-006) y [AGENTS.md](../../AGENTS.md).

| Prompt y entrada | Fase | Etapa | Objetivo | Estado informado en la entrada | Precedente o relacionado | Revisión humana documentada | Commit asociado |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [P-001](P-001.md) | 0 · Preparación | — | Especificar requisitos académicos y del sistema. | Pendiente de ejecución. | — | Lista sin completar. La entrada deja la aprobación pendiente. | Pendiente en la entrada. |
| [P-001.1](P-001.1.md) | 0 · Preparación | — | Consolidar requisitos y resolver ambigüedades. | Pendiente de ejecución. | P-001 | Las 15 resoluciones aprobadas constan en requisitos. La lista de revisión de la ejecución está sin completar. | Pendiente en la entrada. |
| [P-002](P-002.md) | 0 · Preparación | — | Elaborar la arquitectura de referencia y los ADR. | Pendiente de ejecución. | P-001.1 | La revisión de la ejecución está sin completar. La aprobación de las decisiones se documentó después en P-003. | Pendiente en la entrada. |
| [P-003](P-003.md) | 0 · Preparación | — | Formalizar los ADR, registrar controles y redactar AGENTS.md. | Pendiente de ejecución. | P-002 | La aprobación expresa de los 12 ADR figura en los registros. La lista de revisión de la ejecución está sin completar. | Pendiente en la entrada. |
| [P-004](P-004.md) | 0 · Preparación | — | Configurar OpenSpec y documentar el flujo de contribución. | Pendiente de ejecución. | P-003 | Lista sin completar. Los archivos locales respaldan la configuración. | Pendiente en la entrada. |
| P-004.1 · sin entrada propia | 0 · Preparación | — | Continuar P-004 tras la inicialización manual. | Sin estado en bitácora. La configuración final se comprueba en el repositorio. | P-004 | Inicialización manual informada por los responsables. No hay verificación firmada en una entrada propia. | Sin constancia en bitácora. |
| [P-005](P-005.md) | 0 · Preparación | — | Formalizar DEV, organizar el repositorio y cerrar la preparación. | Pendiente de ejecución en la entrada. | P-004 y P-004.1 | P-005 aporta la aprobación expresa de DEV-001 a DEV-010. El cierre requiere revisión del equipo. | Pendiente en la entrada. |

Las seis entradas existentes conservan campos pendientes de respuesta, fecha y revisión. Para completar el historial, incorporar las respuestas y evidencias originales disponibles, los commits cuando consten y el registro de P-004.1 si los responsables disponen de su prompt y respuesta. El [cierre de Fase 0](../preparation/closure.md) reúne la evidencia verificada.

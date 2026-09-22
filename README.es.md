# ia-orchestration-skills

[![Licencia: Apache 2.0](https://img.shields.io/badge/licencia-Apache_2.0-blue.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/formato-Agent%20Skills-8A2BE2.svg)](#anatomía-de-una-skill)

**🇬🇧 [Read this in English](README.md)**

Herramientas de proceso para desarrollo de software aumentado con IA. Aquí no vas a encontrar *prompts* sueltos, sino flujos de trabajo completos que un agente de código ejecuta igual, proyecto tras proyecto.

Este repositorio es la vitrina pública de mi práctica como **AI Orchestrator**. Cuando el agente escribe el 80 % del código, el trabajo de ingeniería se desplaza: ya no consiste en teclear, sino en diseñar el proceso. Qué contexto recibe el agente antes de actuar. En qué orden produce cada artefacto. Cómo se verifica lo que entrega. Y cuánto costó producirlo, de verdad. Cada skill de aquí codifica una de esas decisiones.

## Tabla de contenidos

- [Filosofía](#filosofía)
- [Skills disponibles](#skills-disponibles)
- [Instalación](#instalación)
- [Uso](#uso)
- [Anatomía de una skill](#anatomía-de-una-skill)
- [Compatibilidad](#compatibilidad)
- [Contribuir](#contribuir)
- [Hoja de ruta](#hoja-de-ruta)
- [Licencia](#licencia)
- [Autor](#autor)

---

## Filosofía

**El contexto es el producto.** Un agente sin contexto arquitectónico no escribe código malo. Escribe código plausible que no encaja, que es bastante peor, porque pasa la revisión superficial. Por eso la documentación deja de ser un entregable de fin de proyecto y se convierte en la entrada del sistema.

**Lo que no se mide, se supone.** Trabajar con IA abrió una brecha nueva entre lo que se *siente* rápido y lo que *es* rápido. El estudio de METR de 2025 midió a desarrolladores experimentados un 19 % más lentos con IA, mientras ellos reportaban sentirse un 20 % más rápidos. La diferencia se la come el tiempo de revisión. Y si ese tiempo no es un campo medido, para efectos prácticos no existe.

**Nada de datos inventados.** Preguntarle a un LLM cuántos tokens gastó o cuánto costó una tarea es pedirle justo el dato que no puede conocer y sí puede confabular de manera convincente. La solución no está en escribir un prompt más severo. Está en quitarle el campo de las manos y extraerlo de un artefacto.

---

## Skills disponibles

| Skill | Qué hace |
|---|---|
| **[`project-docs-bootstrap`](skills/project-docs-bootstrap/)** | Crea el sistema de documentación de un proyecto (`CLAUDE.md` → `PRD.md` → `tech-specs.md` → OWASP + git flow → `MEMORY.md` → `TODO.md`) con un **motor JIT** que mantiene siempre exactamente 2 tareas atómicas en el backlog, calculadas comparando el objetivo del producto contra el estado real. |
| **[`ai-effort-tracking`](skills/ai-effort-tracking/)** | Mide el esfuerzo y el **costo real** del desarrollo asistido: tiempo humano frente a tiempo de agente, **peaje de revisión**, tokens y USD por tarea. Funciona en CLI, web, escritorio, móvil y API, con Anthropic, OpenAI, Google, DeepSeek, Qwen y Kimi. |

Las dos se articulan entre sí. `project-docs-bootstrap` produce identificadores estables (`OBJ-3`, `T-0042`) y `ai-effort-tracking` los usa como clave de unión. Juntas responden algo que ninguna herramienta genérica de observabilidad LLM puede responder: **cuánto costó cada objetivo de producto, en dinero y en horas humanas.**

### Qué hace distinto a `ai-effort-tracking`

- **El peaje de revisión, medido.** El hueco entre que el agente termina y que el humano envía el siguiente prompt es tiempo de revisión, y ya está registrado en tu disco. Con hooks pasa de estimado a medido. Y por encima de un umbral configurable se reclasifica como ausencia, para que una sesión que pasa la noche esperando no infle la métrica.
- **Multi-proveedor de verdad.** Anthropic, OpenAI y DeepSeek cuentan los tokens de caché con semánticas incompatibles entre sí. Normalizarlas mal no lanza ningún error: produce cifras presentables y falsas, desviadas por factores de más de 3×. La skill trae el contrato de normalización y las pruebas que lo fijan.
- **Tarifa plana y API, separadas.** Bajo suscripción el costo marginal de un token es cero; bajo API es real. Por eso se registran tres cifras (precio sombra, costo marginal y parte proporcional de la cuota), porque cada una responde una pregunta distinta.
- **Se niega a inventar.** Sin tarifa verificada, el costo queda en `null` y el reporte explica por qué. Un validador rechaza cualquier evento con un campo "medido" que no venga de un extractor.

---

## Instalación

**Para todos tus proyectos:**

```bash
git clone https://github.com/ocastelblanco/ia-orchestration-skills.git
cp -r ia-orchestration-skills/skills/<nombre-de-la-skill> ~/.claude/skills/
```

**Para un solo proyecto** (se versiona junto con el repo):

```bash
cp -r ia-orchestration-skills/skills/<nombre-de-la-skill> .claude/skills/
```

Requisitos: un agente compatible con Agent Skills. `ai-effort-tracking` necesita además **Node.js ≥ 18** para sus scripts, y **Python ≥ 3.10** solo si usas el *wrapper* de API en Python.

---

## Uso

Una skill se activa de dos formas.

La primera es **automática**: el agente detecta que la tarea coincide con la descripción del *frontmatter*. Pedir "documenta este proyecto" dispara `project-docs-bootstrap`; preguntar "cuánto costó esta sesión" dispara `ai-effort-tracking`.

La segunda es **explícita**, invocándola por nombre:

```bash
# Arrancar la documentación de un proyecto
/project-docs-bootstrap

# Configurar el registro de esfuerzo (una vez por proyecto)
/ai-effort-tracking init

# Registrar la unidad de trabajo que acabas de cerrar
/ai-effort-tracking capture

# Reporte del mes
/ai-effort-tracking report
```

Los scripts también corren solos, sin agente de por medio:

```bash
cd skills/ai-effort-tracking

# Migrar un CSV de tracking existente
node scripts/core/ledger.mjs migrate --csv tracking.csv --dir metrics/events --project mi-proyecto

# Extraer tokens, costo y peaje de revisión de una sesión de Claude Code
node scripts/adapters/claude-code-transcript.mjs --project-dir . --aggregate true

# Generar el reporte
node scripts/core/report.mjs --dir metrics/events --from 2026-08-01 --out reporte.md

# Pruebas
node scripts/test.mjs
```

---

## Anatomía de una skill

```
skills/<nombre>/
├── SKILL.md        # Requerido: frontmatter (name, description) + instrucciones
├── references/     # Opcional: material que se carga solo cuando se necesita
├── scripts/        # Opcional: ejecutables que el agente invoca
└── templates/      # Opcional: archivos para copiar en el proyecto destino
```

`SKILL.md` es lo único que el agente carga completo al activarse, así que conviene mantenerlo corto y accionable. El detalle vive en `references/`, que el agente lee solo cuando el propio `SKILL.md` se lo indica.

Ese diseño en capas (metadata siempre visible, cuerpo bajo demanda, referencias bajo demanda) es lo que permite tener decenas de skills instaladas sin saturar la ventana de contexto.

---

## Compatibilidad

Están escritas y probadas con **Claude Code**, en formato estándar de Agent Skills, así que deberían funcionar con cualquier herramienta compatible. Algunos campos del *frontmatter*, como `allowed-tools`, son extensiones específicas de Claude Code y otras herramientas pueden ignorarlos sin problema.

`ai-effort-tracking` mide trabajo hecho con cualquier modelo o herramienta. Lo que cambia de una a otra es cuánta evidencia hay disponible. Los adaptadores de Claude Code están verificados; los de Codex CLI y Gemini CLI están documentados con su fuente identificada y marcados como pendientes de validación empírica. Esa distinción se mantiene visible a propósito.

---

## Contribuir

Los aportes son bienvenidos, sobre todo adaptadores nuevos para otras herramientas de codificación.

1. Crea `skills/<nombre>/SKILL.md` con su *frontmatter*.
2. Mantén `SKILL.md` bajo unas 500 líneas; el detalle va a `references/`.
3. Si añades un adaptador, sigue el [contrato](skills/ai-effort-tracking/references/adapter-contract.md) y **fija un payload real como prueba**, para que un cambio de formato del proveedor falle ruidosamente en vez de en silencio.
4. Marca el estado de verificación con honestidad: *verificado*, *plausible* o *por investigar*.
5. Agrega tu skill a la tabla de ambos README.

Ejecuta `node skills/ai-effort-tracking/scripts/test.mjs` antes de abrir el PR.

---

## Hoja de ruta

- Adaptadores para **Codex CLI** y **Gemini CLI**. Gemini ya emite `gen_ai.client.token.usage`, de la convención GenAI de OpenTelemetry, así que puede alimentar el mismo colector que Claude Code.
- Colector OTLP y tablero en tiempo real, con el registro JSONL como formato de ingesta.
- Estimación asistida: predecir el costo de una tarea nueva a partir del histórico de su tipo.

---

## Licencia

[Apache 2.0](LICENSE). Usa, modifica y redistribuye libremente, también con fines comerciales. Si lo redistribuyes, conserva la licencia y el archivo [NOTICE](NOTICE), e indica qué archivos modificaste. La licencia también te otorga una licencia de patentes de los contribuidores.

---

## Autor

**Oliver Castelblanco** · [@ocastelblanco](https://github.com/ocastelblanco)

Diseño y opero procesos de desarrollo aumentado con IA: sistemas de contexto que evitan alucinaciones, flujos de verificación que detectan lo que el agente no ve, e instrumentación que convierte la intuición sobre productividad en datos que aguantan una revisión.

¿Ideas, correcciones o una herramienta que te gustaría ver aquí? Abre un [issue](https://github.com/ocastelblanco/ia-orchestration-skills/issues).

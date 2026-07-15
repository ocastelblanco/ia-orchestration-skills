# ia-orchestration-skills

Colección pública de *skills* (habilidades) para agentes de código — Claude Code y herramientas compatibles con el estándar de Agent Skills.

Este repositorio es la vitrina pública de una parte de mis herramientas de trabajo como **IA Orchestrator**: en lugar de escribir cada pieza de documentación, planeación o backlog a mano, diseño flujos reutilizables que un agente puede ejecutar de forma consistente, proyecto tras proyecto. Cada skill aquí es un flujo de ese tipo, extraído de mi práctica diaria y empaquetado para que cualquiera lo pueda instalar y usar.

## Tabla de contenidos

- [Filosofía](#filosofía)
- [Anatomía de una skill](#anatomía-de-una-skill)
- [Skills disponibles](#skills-disponibles)
- [Instalación](#instalación)
- [Uso](#uso)
- [Compatibilidad](#compatibilidad)
- [Agregar una nueva skill](#agregar-una-nueva-skill)
- [Roadmap](#roadmap)
- [Licencia](#licencia)
- [Autor](#autor)

---

## Filosofía

Orquestar IA no es solo "escribirle un buen prompt" — es diseñar el proceso completo: qué preguntas debe hacer el agente antes de actuar, en qué orden debe producir cada artefacto, qué formato deben tener las salidas para que otra sesión (u otro agente) pueda retomarlas sin contexto previo, y cómo evitar que el trabajo se convierta en un backlog gigante e inmanejable.

Las skills de este repositorio codifican esas decisiones de proceso, no solo instrucciones sueltas. La primera, [`project-docs-bootstrap`](skills/project-docs-bootstrap/), es el ejemplo más claro: en vez de dejar que el backlog de un proyecto crezca sin control, impone un motor JIT (*just-in-time*) que mantiene siempre exactamente dos tareas atómicas activas, calculadas comparando el objetivo del producto (`PRD.md`) contra el estado real (`MEMORY.md`).

## Anatomía de una skill

Cada skill sigue el formato estándar de Agent Skills:

```
skills/<nombre-de-la-skill>/
├── SKILL.md            # Requerido: frontmatter (name, description) + instrucciones para el agente
└── references/          # Opcional: documentos que el SKILL.md enlaza y que se cargan solo cuando se necesitan
```

- **`SKILL.md`** es lo único que el agente carga siempre que la skill puede aplicar (nombre + descripción) y lo único que carga completo cuando la skill se activa. Por eso se mantiene corto (idealmente bajo 500 líneas) y con instrucciones accionables, no explicaciones largas.
- **`references/`** contiene material de apoyo que el agente solo lee cuando el propio `SKILL.md` lo indica: plantillas para copiar y pegar, guías narrativas extensas, checklists de referencia. Si un archivo de referencia supera ~300 líneas, incluye su propia tabla de contenidos para que el agente pueda saltar directo a la sección relevante.

Este diseño en capas (metadata siempre visible → cuerpo bajo demanda → referencias bajo demanda) es lo que permite tener decenas de skills instaladas sin saturar el contexto del agente.

## Skills disponibles

| Carpeta | Nombre coloquial | Qué hace |
|---|---|---|
| [`project-docs-bootstrap`](skills/project-docs-bootstrap/) | Motor JIT | Crea el sistema de documentación de un proyecto (`CLAUDE.md` → `PRD.md` → `tech-specs.md` → seguridad OWASP + git flow → `MEMORY.md` → `TODO.md`), con un motor JIT que mantiene siempre exactamente 2 tareas atómicas en `TODO.md`. Incluye una guía narrativa completa en [`references/instrucciones-inicio.md`](skills/project-docs-bootstrap/references/instrucciones-inicio.md) para quienes prefieren copiar los prompts manualmente. |

## Instalación

**Para todos tus proyectos** (disponible en cualquier sesión de Claude Code):
```bash
cp -r skills/<nombre-de-la-skill> ~/.claude/skills/
```

**Para un solo proyecto** (se versiona junto con el repo del proyecto):
```bash
cp -r skills/<nombre-de-la-skill> .claude/skills/
```

## Uso

Una vez instalada, una skill se activa de dos formas:

1. **Automática** — el agente detecta que la tarea coincide con la `description` del frontmatter y consulta la skill sin que se lo pidas explícitamente. Por ejemplo, pedir "documenta este proyecto" o "crea el PRD" activa `project-docs-bootstrap`.
2. **Explícita** — la invocas por nombre: `/project-docs-bootstrap`.

## Compatibilidad

Estas skills están escritas y probadas principalmente con **Claude Code**. Usan el formato estándar `SKILL.md` (frontmatter `name` + `description`, cuerpo en Markdown, `references/` opcional), por lo que en principio deberían funcionar con cualquier herramienta compatible con el estándar de Agent Skills. Algunos campos del frontmatter, como `allowed-tools`, son extensiones específicas de Claude Code — otras herramientas pueden ignorarlos sin problema.

## Agregar una nueva skill

1. Crear `skills/<nombre-de-la-skill>/SKILL.md` con el frontmatter (`name`, `description` y, si aplica, `allowed-tools`).
2. Agregar `references/` con plantillas o guías complementarias si la skill las necesita.
3. Añadir una fila a la tabla de [skills disponibles](#skills-disponibles) en este README.
4. Mantener `SKILL.md` corto y accionable — si crece más allá de ~500 líneas, mover el detalle a `references/` y dejar en `SKILL.md` solo el flujo y un puntero claro a dónde ir.

## Roadmap

Este es el primer paso de una colección más grande. La intención es seguir extrayendo flujos de mi práctica de orquestación de IA — planeación, revisión de código, despliegue, gestión de backlog — y publicarlos aquí a medida que se estabilizan.

## Licencia

[MIT](LICENSE) — usa, copia, modifica y redistribuye estas skills libremente, con o sin atribución.

## Autor

Mantenido por [Oliver Castelblanco](https://github.com/ocastelblanco) como parte de su práctica de orquestación de agentes de IA para desarrollo de software.

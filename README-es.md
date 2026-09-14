[English](README.md) · [Español](README-es.md)

<p align="center">
  <h1 align="center">Spec-Driven Skills para Claude Code</h1>
  <p align="center">Planifica la feature. Apruébala. Impleméntala por grupos.</p>
</p>

<p align="center">
  <img alt="Licencia" src="https://img.shields.io/github/license/elmerjacobo97/spec-flow-skills">
  <img alt="Último release" src="https://img.shields.io/github/v/release/elmerjacobo97/spec-flow-skills">
  <img alt="GitHub Stars" src="https://img.shields.io/github/stars/elmerjacobo97/spec-flow-skills?style=social">
  <img alt="Skills" src="https://img.shields.io/badge/skills-7-blue">
</p>

## Inicio rápido

```bash
npx skills@latest add elmerjacobo97/spec-flow-skills
```

## Skills

| Skill | Descripción | Argumento |
| --- | --- | --- |
| `/spec-explore` | Piensa una idea difusa contra el código real — opciones, tradeoffs, sin artefactos | `[tema]` |
| `/product-spec` | Define el caso de negocio antes del spec técnico: evidencia, métrica, versión 20%, criterio de kill | `[tema corto]` |
| `/spec` | Diseña el documento de la feature haciendo preguntas de clarificación | — |
| `/spec-edit` | Edita un spec existente en el lugar: análisis de impacto, un preview, un solo diff mínimo | `<NN-slug> [cambio]` |
| `/spec-impl` | Valida que el spec esté aprobado y lo implementa por grupos, pausando para revisar y commitear tras cada grupo | `<NN-slug> [--one-shot]` |
| `/spec-status` | Tablero read-only de todos los specs: estado, progreso, dependencias, próxima acción | — |
| `/spec-verify` | Audita spec ↔ código: completitud, corrección, coherencia; reporta, nunca arregla | `<NN-slug>` |

---

## Tabla de contenidos

- [Qué es spec-driven design](#qué-es-spec-driven-design)
- [El problema que resuelve](#el-problema-que-resuelve)
- [El procedimiento en seis pasos](#el-procedimiento-en-seis-pasos)
- [Anatomía de un spec útil](#anatomía-de-un-spec-útil)
- [El paso opcional antes de /spec: /product-spec](#el-paso-opcional-antes-de-spec-product-spec)
- [Cuándo SÍ y cuándo NO usar specs](#cuándo-sí-y-cuándo-no-usar-specs)
- [Reglas que casi nadie sigue](#reglas-que-casi-nadie-sigue)
- [Instalación](#instalación)
- [Uso](#uso)

---

## Qué es spec-driven design

Spec-driven design es un enfoque donde **el spec es el artefacto principal del trabajo, no el código**. El código es la consecuencia.

Suena obvio. La diferencia con "documentar antes de programar" de toda la vida es que en spec-driven el spec **no es opcional ni decorativo**: es el contrato que guía la ejecución, se versiona en git, y se mantiene vivo. Si el código diverge del spec, uno de los dos está mal.

Cada spec captura las decisiones de una sola feature. Los specs viven en `specs/` como archivos `.md` numerados secuencialmente, y forman el registro de decisiones de diseño del proyecto.

---

## El problema que resuelve

Cuando trabajas con un LLM como Claude Code, hay un fenómeno muy concreto: si le pides _"hazme un Arkanoid con power-ups y niveles"_, **va a improvisar**. Va a tomar 50 decisiones de diseño implícitas (¿clases o funciones? ¿estado global o local? ¿cómo se nombran las entidades?) sin que tú las veas. Y cada una de esas decisiones se vuelve un acoplamiento caro de revertir después.

El problema no es nuevo — los humanos también improvisan — pero con un LLM es más agudo:

1. **La velocidad de generación oculta el costo de las decisiones.** Cuando un humano tarda dos horas en escribir un módulo, tiene tiempo de pensar. Cuando Claude lo hace en 30 segundos, las decisiones pasan invisibles.
2. **Cada conversación arranca de cero.** Sin un spec, en la siguiente sesión Claude no sabe qué decidiste antes y va a improvisar otra vez, posiblemente en dirección contraria.
3. **El contexto se llena rápido.** Sin un documento estable al que referirse, terminas pegando contexto a mano en cada prompt.

El spec resuelve los tres: hace explícitas las decisiones, persiste entre sesiones, y se carga una vez como referencia.

---

## El procedimiento en seis pasos

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  1. DESCRIBIR   │→ │  2. PLAN MODE   │→ │   3. REFINAR    │
│  el problema    │  │ Claude propone  │  │ Tú das          │
│  no la solución │  │ no edita        │  │ decisiones      │
└─────────────────┘  └─────────────────┘  └─────────────────┘
        ↑                                          │
        │                                          │
        └──────── 2-3 iteraciones hasta converger ─┘
                              │
                              ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   4. GUARDAR    │→ │   5. EJECUTAR   │→ │   6. REVISAR    │
│ specs/NN-       │  │ Por grupos,     │  │ Diff + commit   │
│ feature.md      │  │ boxes marcados  │  │ por grupo       │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### 1. Describir

Le describes a Claude la feature en términos de **problema**, no de solución. Si dictas la solución, Claude solo la formatea — pierdes su capacidad de proponer estructura.

### 2. Plan mode

Activas plan mode (en plan mode Claude no puede escribir archivos, solo leer y proponer). Claude responde con un documento estructurado: alcance, modelo de datos, plan de implementación y criterios de aceptación.

### 3. Refinar

Lees el plan con resistencia y das **decisiones concretas**. "Saca X del alcance", "los datos viven en JSON, no en módulos JS", "añade una sección de riesgos". Iteras 2-3 veces.

### 4. Guardar

Cuando el spec está afinado, lo guardas en `specs/NN-slug.md` con estado `Borrador`. Sales del chat, **lo relees fuera del editor**, y solo cuando estás conforme cambias el estado a `Aprobado` manualmente. Ese cambio lo hace el humano, no Claude.

### 5. Ejecutar

Sales de plan mode y corres `/spec-impl`. Claude implementa un grupo del plan, marca cada paso dentro del archivo del spec y se detiene con un mini-resumen. Revisas el diff y commiteas. Dices "continúa" y pasa al siguiente grupo. Solo se detiene a mitad de grupo ante una ambigüedad real o un paso que rompe algo. Para specs chicos, agrega `--one-shot` y corre todos los grupos sin pausas.

### 6. Revisar

Revisas y commiteas por grupo: cada trozo es suficientemente chico para leerlo de verdad, y el historial queda limpio. Cuando termina el último grupo, Claude verifica los criterios de aceptación con evidencia real y cierra con un resumen: qué se hizo, por qué, qué quedó verificado y qué falta. Interrumpir es barato — los checkboxes sin marcar le dicen a la siguiente corrida exactamente dónde reanudar.

---

## Anatomía de un spec útil

No todos los documentos sirven. Un spec útil tiene seis partes — si falta alguna, probablemente no es suficiente para guiar la ejecución.

### 1. Objetivo en una frase

Si no cabe en una frase, la feature es demasiado grande. Divídela antes de escribir nada más.

### 2. Alcance explícito + lo que NO entra

El "no entra" es tan importante como el "entra". Sin él, los límites son borrosos y aparece scope creep durante la implementación. Captura las cosas que se mencionaron pero se decidió posponer.

### 3. Modelo de datos

Las estructuras y nombres concretos. Si dices "el módulo de niveles", di `src/levels.js`. Si dices "una clave", da el string exacto. Esta sección es la que más se cita después en otros specs y skills.

### 4. Plan de implementación ordenado

Pasos en checkboxes agrupados (`### Grupo N` + `- [ ] N.M`). **Cada paso debe dejar el sistema en estado funcional.** Si un paso requiere más de 30-50 líneas de código, divídelo. El último paso no es "probar todo" — eso son los criterios de aceptación. `/spec-impl` marca cada paso al implementarlo y se detiene al final de cada grupo para que revises y commitees antes del siguiente.

### 5. Criterios de aceptación

Checklist booleano verificable. Cada item se puede contestar con sí o no.

- ❌ "Que funcione bien" — no es verificable
- ❌ "Buena UX" — subjetivo
- ❌ "Sin bugs" — no operacional
- ✅ "Pulsar Esc pausa el juego y muestra el menú" — verificable

### 6. Decisiones tomadas y descartadas

Lo que consideraste y por qué elegiste lo que elegiste. **Esto es oro dentro de tres meses** cuando alguien pregunte _"¿por qué la persistencia usa una key versionada?"_. La respuesta vive ahí.

Cada decisión idealmente tiene una razón breve. Las decisiones sin razón son las primeras que se cuestionan después.

---

## El paso opcional antes de /spec: /product-spec

`/spec` diseña *qué* se construye una vez que alguien ya decidió que vale la pena construirlo. No pregunta *si* vale la pena ni *cómo vas a saber* si funcionó — esa pregunta es de `/product-spec`, un skill separado que corre un paso antes.

```bash
/product-spec notificaciones-push   # opcional — evidencia, métrica, versión 20%, criterio de kill
        ↓
/spec notificaciones-push           # lee el brief automáticamente si existe
        ↓
/spec-impl NN-notificaciones-push
```

`/product-spec` hace cinco preguntas en una sola pasada — quién lo pidió, qué hacen hoy sin esto, qué métrica debería mover (con baseline real, no un adjetivo), cuál es la versión más chica que prueba la hipótesis, y qué número te haría matarlo — y guarda `specs/NN-slug.brief.md`. Si no hay baseline contra qué medir, no inventa uno: entrega un brief que dice qué instrumentar primero, y se detiene ahí.

**Úsalo cuando** el valor o la audiencia de la feature no es ya obvio. **Sáltalo** en bug fixes, parches de seguridad, deuda técnica, o cualquier cosa que un cliente ya pidió con evidencia dura — en esos casos ve directo a `/spec`.

Ver [`skills/product/product-spec/SKILL.md`](./skills/product/product-spec/SKILL.md) para el flujo completo.

---

## Cuándo SÍ y cuándo NO usar specs

Esta arquitectura tiene costo. No la apliques a todo.

### SÍ — escribe un spec cuando:

- La tarea tocará **más de dos archivos**.
- Hay **decisiones caras de revertir** (esquemas de datos, formatos, APIs).
- La feature ocupará **más de una sesión** de Claude Code.
- Existe un **contrato que otros artefactos van a reusar** (otro spec, un skill, un hook).
- Es algo que **vas a olvidar en una semana**.

### NO — usa un prompt directo cuando:

- Es un **bug fix puntual**.
- Es un **refactor mecánico** (renombrar, mover archivos).
- Es un **experimento exploratorio** donde el objetivo es descubrir la decisión, no ejecutarla.
- La tarea **cabe en un prompt** y se entiende a la primera.
- Es una **tarea única** que no se va a repetir.

### Regla mental

> **Si te tienta abrir plan mode, probablemente lo necesites.** > **Si la feature te aburre planearla, probablemente no.**

El sentido común vence a la regla — pero el sentido común se entrena con las dos columnas de arriba.

---

## Reglas que casi nadie sigue

Cuatro patrones de uso que distinguen al método funcionando bien del método como burocracia decorativa:

### 1. En la fase de descripción, describe el problema, no la solución

❌ _"Añade un array de niveles cargados desde JSON, una función `loadLevel()`, y persistencia con localStorage versionado."_

Eso ya es un spec mal escrito por ti. Claude solo lo va a formatear.

✅ _"Quiero que el juego deje de ser de una sola pantalla. La siguiente feature es: progresión por niveles con dificultad creciente, y persistencia de mejores puntajes entre sesiones."_

Esa segunda versión deja espacio para que Claude **decida** y tú **revises**. Esa es la naturaleza del flujo.

### 2. En la fase de refinar, da decisiones concretas, no sugerencias

Plan mode es donde **tú diriges**. "Saca X", "el formato es JSON", "añade riesgos". Si dices "creo que tal vez sería bueno...", Claude va a dejarlo como está.

### 3. En la ejecución, revisa por grupos — ni por paso ni todo de golpe

- **Muy lento:** pausar tras cada paso te vuelve niñera de una corrida que pudo ser una sola sentada.
- **Muy rápido:** una sola pasada sobre un spec grande termina en un diff gigante, y un error del grupo 2 queda enterrado bajo los grupos 3 y 4.
- **El balance:** los grupos del spec son la unidad de revisión. Claude implementa un grupo, marca sus pasos dentro del archivo del spec y se detiene. Lees el diff, commiteas, dices "continúa". Cada trozo es suficientemente chico para revisarlo de verdad y el historial queda limpio. Una corrida interrumpida reanuda desde el primer box sin marcar — incluso en una sesión nueva.

### 4. Si a mitad de la ejecución quieres cambiar algo, vuelves al paso 2 — nunca improvisas

A mitad de implementación se te ocurre algo. Lo correcto es: parar, volver a plan mode, actualizar el spec, salir, seguir. **No improvisar sobre el código.**

Esa separación es lo que evita el scope creep silencioso.

---

## Instalación

### Opción 1 — skills.sh (recomendado, Claude Code)

```bash
npx skills@latest add elmerjacobo97/spec-flow-skills
```

Para desinstalar:

```bash
npx skills@latest remove elmerjacobo97/spec-flow-skills
```


### Opción 2 — Otros agentes (Cursor, Codex, Antigravity, opencode)

```bash
git clone https://github.com/elmerjacobo97/spec-flow-skills ~/.spec-flow-skills
cd ~/tu-proyecto
~/.spec-flow-skills/scripts/install-to-agent.sh <agent>
```

`<agent>` puede ser `claude`, `cursor`, `codex`, `antigravity` u `opencode`. Ver [README.md](./README.md#installation) para detalles.

### Opción 3 — Manual

```bash
# Personal (todos tus proyectos)
mkdir -p ~/.claude/skills
cp -r skills/product/product-spec ~/.claude/skills/
cp -r skills/engineering/spec ~/.claude/skills/
cp -r skills/engineering/spec-impl ~/.claude/skills/

# O de proyecto (versionado en git)
mkdir -p .claude/skills
cp -r skills/product/product-spec .claude/skills/
cp -r skills/engineering/spec .claude/skills/
cp -r skills/engineering/spec-impl .claude/skills/
```

Para que el método funcione, también necesitas crear la carpeta `specs/` en la raíz del proyecto:

```bash
mkdir specs
```

Opcionalmente, añade un `specs/README.md` que documente la convención (ver el ejemplo en este repo).

---

## Uso

### Ciclo completo de una feature

```bash
# 0. (Opcional) ¿No sabes aún cómo abordarlo? Explora primero — no crea archivos
/spec-explore niveles-y-highscores

# 0b. (Opcional) Define el caso de negocio — evidencia, métrica, criterio de kill
/product-spec niveles-y-highscores

# Solo vale la pena si el valor o la audiencia de la feature no es ya obvio.
# Guarda specs/03-niveles-y-highscores.brief.md con estado: Borrador.

# 1. Diseñar el spec con preguntas de clarificación
/spec niveles-y-highscores

# Claude lee el archivo de memoria del proyecto (CLAUDE.md, AGENTS.md, GEMINI.md o README.md) y specs/ existentes
# (incluyendo el .brief.md del paso 0, si existe), hace preguntas
# en bloques, desarrolla el spec sección por sección,
# y al final lo guarda como specs/03-niveles-y-highscores.md
# con estado: Borrador.

# 2. Releer el spec fuera del chat y aprobarlo manualmente
# (abrir el archivo en el editor, cambiar Estado: Borrador → Aprobado)

# 3. Implementar el spec aprobado
/spec-impl 03-niveles-y-highscores

# Claude valida que el estado sea Aprobado y resuelve la rama:
# reutiliza la rama de ticket activa si existe; si no, crea
# spec-03-niveles-y-highscores. Implementa un grupo, lo marca en
# el spec y se detiene para que revises y commitees. Dices
# "continúa" para el siguiente grupo. Agrega --one-shot para
# saltarte las pausas en specs chicos.

# 4. (Opcional) Audita que el código coincida con el spec
/spec-verify 03-niveles-y-highscores

# En cualquier momento: /spec-status muestra todos los specs — estado, progreso, dependencias.
```

### Qué hace cada skill

#### `/spec-explore [tema]`

Un partner de pensamiento sin stakes para ideas que aún no están listas para un spec:

1. **Investiga** — lee el código relevante y cita lo que existe hoy.
2. **Opciones** — presenta 2-3 enfoques concretos con tradeoffs y una recomendación.
3. **Itera** — profundiza turno a turno mientras la idea se afina.
4. **Handoff** — cuando la idea ya se puede delimitar, apunta a `/product-spec` (valor aún difuso) o `/spec` (lista). No crea nada.

#### `/product-spec [tema-corto]`

Define el caso de negocio, en una sola pasada:

1. **Evidencia** — quién lo pidió, cuántos, qué tan fuerte es la señal.
2. **Métrica** — baseline, target y fecha de chequeo. No acepta adjetivos.
3. **Versión 20%** — el corte más chico que igual prueba la hipótesis.
4. **Criterio de kill** — qué resultado revierte esto en vez de iterar sobre ello.
5. **Guardar** — `specs/NN-slug.brief.md` con estado `Borrador`. Si no hay baseline que medir, el brief recomienda instrumentación en vez de definir el alcance de una feature.

#### `/spec [tema-corto]`

Diseña el documento de la feature. Pasa por cuatro fases:

1. **Contexto** — lee el archivo de memoria del proyecto (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md` o `README.md`, el primero que exista) y los specs previos.
2. **Clarificación** — hace preguntas en bloques de 3-5 hasta que la feature está claramente definida.
3. **Desarrollo sección por sección** — genera y confirma cada sección del spec antes de pasar a la siguiente.
4. **Guardar** — escribe el archivo en `specs/NN-slug.md` con estado `Borrador`.

#### `/spec-edit <NN-nombre> [qué-cambiar]`

Revisa un spec existente en el lugar — sin regenerar, sin archivo nuevo. Pasa por este flujo:

1. **Identificar** — localiza el archivo (mismo matching flexible que `/spec-impl`).
2. **Leer e indexar** — secciones por significado, estado actual, qué pasos ya están marcados.
3. **Entender el cambio** — si tu pedido ya es claro salta directo; si no, hace un bloque de 2–4 preguntas concretas. También aplica la regla editar-vs-spec-nuevo: mismo trabajo refinado → edita; si la intención cambió o el scope explotó → recomienda un `/spec` nuevo.
4. **Análisis de impacto y preview** — muestra las ediciones exactas (`viejo → nuevo`) por sección, más las cascadas (cambio de scope → plan y criterios) y cualquier conflicto, como tocar un paso ya marcado `- [x]`.
5. **Una confirmación → aplicar** — diffs mínimos, idioma y formato del spec intactos, fecha del header actualizada. La línea de estado nunca se toca.
6. **Chequeo de coherencia y resumen** — reporta lo que quede flojo y avisa cuando el spec necesita re-aprobación humana (estaba `Aprobado`) o puede divergir del código (estaba `Implementado`).

> **El estado sigue siendo humano:** `/spec-edit` nunca edita la línea `Estado:` / `Status:`. Si el spec estaba `Aprobado` antes de la edición, re-apruébalo a mano cuando estés conforme — `/spec-impl` solo trabaja con un spec aprobado.

#### `/spec-impl <NN-nombre>`

Implementa un spec aprobado. Pasa por cuatro fases:

1. **Identificar** — busca el archivo del spec.
2. **Validar** — verifica que el estado sea `Aprobado`. Si no, se detiene.
3. **Resolver rama** — reutiliza la rama de ticket activa (`feat/...`, `fix/...`) si existe; si no, crea y se mueve a `spec-NN-slug`.
4. **Implementar** — por grupos. Marca cada paso `- [x]` dentro del spec al completar un grupo y se detiene con un mini-resumen para que revises y commitees. Dices "continúa" para el siguiente grupo. Verifica los criterios de aceptación con evidencia real al final y cierra con un resumen. Solo se detiene a mitad de grupo ante una ambigüedad o un paso que rompe el proyecto.

> **Los grupos viven en el spec:** el plan de implementación es una lista de checkboxes agrupada (`### Grupo N — …` + `- [ ] N.M`). Si un spec viejo está plano, `/spec-impl` propone una agrupación, la escribe en el spec tras tu confirmación y luego implementa grupo por grupo. Una corrida interrumpida reanuda desde el primer box sin marcar, y al final solo se marcan los criterios que se pudieron probar con evidencia — el resto te espera.

> **`--one-shot`:** `/spec-impl 03-niveles-y-highscores --one-shot` corre todos los grupos sin las pausas intermedias. Útil cuando el spec es chico y basta con una revisión al final.

> **Control de la rama:** La Fase 3 primero revisa la rama actual. Si no es la default ni `spec-NN-slug` — el caso cuando una herramienta de tickets (p.ej. Forge) ya creó la rama de trabajo — la conserva y omite `AutoCreateBranch` por completo: una rama de trabajo por tarea, creada una sola vez. En caso contrario lee el flag `AutoCreateBranch` de `specs/.spec-config.yml`. Por defecto es `true` (crea la rama automáticamente). Ponlo en `false` para que `/spec-impl` pregunte `[s/N]` antes de crear cualquier rama — útil si el nombrado de ramas es parte de tu propio Git workflow.
>
> ```yaml
> # specs/.spec-config.yml
> AutoCreateBranch: false
> ```

#### `/spec-status`

Tablero read-only de todo lo que vive en `specs/`:

1. **Inventario** — specs, briefs de producto y config, separados.
2. **Lectura** — estado, dependencias y conteo de checkboxes de plan/criterios por spec.
3. **Tablero** — una tabla ordenada por número, con banderas: `Implementado` con criterios sin marcar, dependencia que no está `Implementado`, header con estado roto.
4. **Próximas acciones** — una acción sugerida por spec pendiente. Nunca las ejecuta.

#### `/spec-verify <NN-nombre>`

Auditoría independiente de una implementación contra su spec — re-ejecutable en cualquier momento, incluso meses después para detectar drift:

1. **Indexa** — pasos del plan, criterios, decisiones y modelo de datos.
2. **Evidencia** — busca en el código cada afirmación (un checkbox es una afirmación, no evidencia) y corre tests/build/lint si el proyecto los define.
3. **Reporta** — completitud, corrección, coherencia; cada hallazgo etiquetado CRITICAL / WARNING / SUGGESTION con `file:line`.
4. **Veredicto** — más la dirección del desajuste: spec viejo → `/spec-edit`; código drift → arreglar el código. Es read-only: nunca arregla nada por su cuenta.

### Estados de un spec

| Estado         | Significado                                                              |
| -------------- | ------------------------------------------------------------------------ |
| `Borrador`     | El skill `/spec` lo generó pero el humano no lo ha releído.              |
| `En revisión`  | El humano lo está revisando o iterando con Claude.                       |
| `Aprobado`     | El humano lo leyó y autorizó. `/spec-impl` solo trabaja con este estado. |
| `Implementado` | El código existe y pasa los criterios de aceptación.                     |
| `Obsoleto`     | Reemplazado por otro spec. No se borra — se referencia.                  |

**Cambiar el estado a `Aprobado` es un acto humano deliberado.** Es la única firma del contrato — Claude no puede aprobar su propio trabajo.

---

## Por qué los dos skills funcionan como pareja

```
┌───────────────────────────────────────────────────────────┐
│                                                           │
│   /spec     Claude pregunta y diseña                      │
│             ↓                                             │
│             specs/NN-slug.md  (Estado: Borrador)          │
│                                                           │
│   ──────── humano relee y aprueba ────────                │
│             ↓                                             │
│             specs/NN-slug.md  (Estado: Aprobado)          │
│                                                           │
│   /spec-impl  Claude valida e implementa                  │
│             ↓                                             │
│             rama spec-NN-slug (o rama de ticket) + código │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

El gap entre los dos skills — releer y cambiar el estado a mano — es deliberado. Es el único momento donde **solo tú puedes hacer algo**. Sin ese gap, el método se degrada a "Claude escribe documentación bonita y luego escribe el código que se le ocurra de todos modos".

Si el spec necesita cambios antes o después de aprobarse, `/spec-edit` lo revisa en el lugar — y el humano lo vuelve a aprobar antes de que la implementación continúe.

---

## Releases

Este proyecto usa [release-please](https://github.com/googleapis/release-please) para releases automáticos. Los mensajes de commit deben seguir [Conventional Commits](https://www.conventionalcommits.org/):

| Prefijo | Efecto |
| --- | --- |
| `feat:` | Sube versión minor |
| `fix:` | Sube versión patch |
| `feat!:` / `fix!:` | Sube versión major |
| `docs:`, `chore:`, `refactor:` | Sin cambio de versión |

---

## Licencia

MIT

---

_Si encuentras una forma de mejorar el método o los skills, abre un issue o un PR. La parte más valiosa de un skill personal es que evoluciona con el uso._

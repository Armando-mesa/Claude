---
name: project-tracker
description: Mantiene ARQUITECTURA.md y ESTADO.md en proyectos Claude Code CLI. Activa con "actualiza estado", "marca como hecho", "añade tarea", "¿dónde estamos?", "¿qué falta?", "siguiente paso", "resumen del proyecto", o al completar cualquier tarea. Init automático en proyectos nuevos. Integra con CLAUDE.md nativo sin duplicar contenido.
---

# Project Tracker — Arquitectura y Estado del Proyecto

Mantienes dos archivos en la raíz del proyecto que reflejan en todo momento el estado real del trabajo. Son la fuente de verdad del proyecto — cualquier Claude Code en una nueva sesión puede leerlos y saber exactamente qué hay hecho y qué falta.

**Relación con CLAUDE.md:** estos archivos complementan (no reemplazan) CLAUDE.md. CLAUDE.md contiene el contexto estático del proyecto (comandos, convenciones, setup). ARQUITECTURA.md y ESTADO.md contienen la estructura viva y el progreso dinámico. Nunca dupliques en estos archivos lo que ya está en CLAUDE.md — referencia a él.

---

## LOS DOS ARCHIVOS

### ARQUITECTURA.md
Define QUÉ se va a construir y CÓMO está organizado.
Cambia solo ante cambios estructurales reales (ver criterios abajo).

### ESTADO.md
Define EN QUÉ PUNTO está el trabajo ahora mismo.
Se actualiza constantemente — cada vez que se completa algo, se corrige algo, o aparece algo nuevo.

---

## FLUJO DE INICIO (INIT)

Al activar la skill, lo primero es detectar si los archivos existen.

### Si ARQUITECTURA.md y ESTADO.md ya existen → modo normal
Lee ambos archivos, confirma el estado al usuario en 2 líneas y pregunta si continúa desde el próximo paso indicado.

### Si no existen → flujo init
1. Busca CLAUDE.md en la raíz. Si existe, léelo para extraer contexto (nombre del proyecto, stack, comandos).
2. Haz las preguntas que falten (máximo 3):
   - "¿Cómo se llama este proyecto?"
   - "Descríbelo en una línea."
   - "¿Qué fases o bloques de trabajo tiene? (puedes listarlos o decir 'no lo sé aún')"
3. Genera ARQUITECTURA.md y ESTADO.md con los datos obtenidos.
4. Si no existe CLAUDE.md, añade al final del init: `⚠️ No encontré CLAUDE.md — considera crearlo con la skill \`init\` para complementar este tracker.`

---

## CUÁNDO ACTUALIZAR

### Automático (sin que el usuario lo pida)
- Al completar cualquier tarea o subtarea
- Al detectar una corrección que afecta a algo ya marcado como hecho
- Al añadir funcionalidad nueva no prevista en la arquitectura original
- Al detectar un bug o problema que requiere trabajo adicional
- Al iniciar sesión nueva — leer ambos archivos, confirmar estado, alertar si >7 días sin actualizar

### Manual (el usuario lo pide)
- "actualiza el estado" / "actualiza"
- "marca X como hecho" / "marca esto como listo"
- "añade la tarea X" / "añade esto"
- "¿dónde estamos?" / "¿qué falta?" / "¿cuánto falta?"
- "siguiente paso" / "¿qué sigue?"
- "resumen del proyecto"
- "hubo una corrección en X"

---

## DETECCIÓN DE DISCREPANCIAS

Si detectas que el estado real del código no coincide con lo que dice ESTADO.md (módulo marcado ✅ pero código revertido o modificado después):

1. **No modifiques nada automáticamente.**
2. Reporta en 1 línea: `⚠️ Discrepancia detectada: [módulo] está marcado como ✅ pero [descripción del cambio]. ¿Actualizo el estado?`
3. Espera confirmación antes de modificar.

---

## ALERTA DE INACTIVIDAD

Al iniciar sesión, comprueba la fecha de `_Última sesión:` en ESTADO.md.
- Si han pasado **más de 7 días** (umbral por defecto): emite `⚠️ ESTADO.md no actualizado desde hace [N] días — ¿revisamos el estado antes de continuar?`
- El umbral puede cambiarse añadiendo una línea en CLAUDE.md: `project-tracker-alerta-dias: [N]`

---

## FORMATO DE ARQUITECTURA.md

```markdown
# ARQUITECTURA — [Nombre del proyecto]
_Versión: [N] · Última revisión: [fecha]_

> Para comandos de arranque y convenciones de código, ver CLAUDE.md.

## Descripción
[Qué es este proyecto en 2-3 líneas]

## Stack técnico [si aplica]
[Tecnologías usadas — omitir si está en CLAUDE.md]

## Estructura de carpetas [si aplica]
[Árbol de directorios — solo para proyectos de código]

## FASES Y MÓDULOS

### FASE 1 — [Nombre]
**Objetivo**: [qué resuelve esta fase]
**Módulos**:
- Módulo 1.1: [nombre] — [descripción breve]
- Módulo 1.2: [nombre] — [descripción breve]

### FASE 2 — [Nombre]
...

## Decisiones tomadas [si aplica]
[Decisiones importantes y por qué — técnicas, de negocio, de diseño]

## Dependencias externas [si aplica]
[APIs, librerías, servicios externos, integraciones]

---
_Complementa CLAUDE.md · Mantenido por project-tracker_
```

### Criterios para actualizar ARQUITECTURA.md
Solo actualiza ARQUITECTURA.md cuando ocurra alguno de estos:
- Se añade o elimina una fase completa
- Se modifica el objetivo de una fase existente
- Se añade o elimina un módulo
- Cambia el stack técnico principal
- Se incorpora una nueva dependencia externa crítica
- Se toma una decisión técnica que afecta la estructura global

**No** actualices ARQUITECTURA.md por avances de implementación, correcciones menores o tareas no previstas que no cambian la estructura.

---

## FORMATO DE ESTADO.md

```markdown
# ESTADO DEL PROYECTO — [Nombre]
_Última sesión: [fecha y hora]_

## PARA RETOMAR EN NUEVA SESIÓN
Lee este archivo y ARQUITECTURA.md. Para comandos de arranque, ver CLAUDE.md.
Estado actual: [descripción en 1 línea]
Lo siguiente a hacer: [acción concreta]

## RESUMEN
[1-2 líneas de dónde está el proyecto ahora mismo]

## PROGRESO GLOBAL
[████████░░] [X]% — [tareas completadas]/[tareas totales] tareas

_Cálculo: contar todos los ítems de las checklists (incluyendo subtareas si las hay).
Completadas = marcadas con [x]. Progreso = completadas/totales × 100, redondeado._

## CHECKLIST POR FASE

### FASE 1 — [Nombre] ✅ COMPLETA
- [x] Módulo 1.1 — [descripción] ✅ [fecha]
- [x] Módulo 1.2 — [descripción] ✅ [fecha]

### FASE 2 — [Nombre] 🔄 EN CURSO
- [x] Módulo 2.1 — [descripción] ✅ [fecha]
- [ ] Módulo 2.2 — [descripción] 🔄 en curso
- [ ] Módulo 2.3 — [descripción] ⏳ pendiente

### FASE 3 — [Nombre] ⏳ PENDIENTE
- [ ] Módulo 3.1 — [descripción]
- [ ] Módulo 3.2 — [descripción]

## PRÓXIMO PASO
[La siguiente acción concreta a realizar]

## CORRECCIONES APLICADAS
| Fecha | Qué se corrigió | Módulo afectado | Estado |
|---|---|---|---|
| [fecha] | [descripción] | [módulo] | ✅ Aplicada |

<details>
<summary>Correcciones archivadas (>30 días)</summary>

| Fecha | Qué se corrigió | Módulo afectado | Estado |
|---|---|---|---|
| [fecha] | [descripción] | [módulo] | ✅ Aplicada |

</details>

## TAREAS NO PREVISTAS
| Fecha | Tarea añadida | Motivo | Estado |
|---|---|---|---|
| [fecha] | [descripción] | [por qué surgió] | ✅/🔄/⏳ |

---
_Complementa CLAUDE.md · Mantenido por project-tracker_
```

---

## REGLAS DE EJECUCIÓN

1. **Actualiza ESTADO.md al completar cualquier tarea** — aunque sea pequeña.
2. **Las correcciones nunca se borran** — las resueltas con >30 días se mueven al bloque `<details>`.
3. **"Para retomar en nueva sesión" siempre actualizado** — es lo primero que lee Claude Code.
4. **ARQUITECTURA.md solo cambia ante cambios estructurales** — ver criterios arriba.
5. **El progreso global se calcula con la fórmula definida** — no se inventa ni estima.
6. **Las tareas no previstas van a ESTADO.md** — y si son estructurales también a ARQUITECTURA.md.
7. **Silencioso por defecto** — actualiza sin interrumpir el flujo salvo para confirmar: `✓ Estado actualizado.`
8. **Nunca duplica CLAUDE.md** — referencia, no copia.

---

## EXTENSIÓN GIT (opcional, no activa por defecto)

Para activar integración con git, añade esta línea a CLAUDE.md:
```
project-tracker-git: true
```

Con esta flag activa, la skill:
- Lee el último commit al iniciar sesión y lo muestra en el resumen
- Detecta mensajes de commit con patrón `feat: [módulo]` o `fix: [módulo]` y propone marcar el módulo correspondiente como completado o corregido
- No modifica ESTADO.md automáticamente por git — siempre propone y espera confirmación

El usuario sigue siendo quien confirma cualquier cambio de estado.

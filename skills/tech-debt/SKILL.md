---
name: tech-debt
description: Identify, categorize, and prioritize technical debt. Trigger with "tech debt", "technical debt audit", "what should we refactor", "code health", "reorganizar", "deuda técnica", "qué está mal", "¿qué refactorizamos?", or when the user asks about code quality, refactoring priorities, or maintenance backlog.
---

# Tech Debt Management

Systematically identify, categorize, and prioritize technical debt — without breaking what already works.

---

## REGLAS DE ORO (leer antes de cualquier análisis)

1. **Verificar FROZEN_FILES.md primero** — si existe en el proyecto, ningún archivo listado ahí puede recibir recomendaciones de refactor sin aprobación explícita del usuario.
2. **Nunca proponer cambios al bot/servicio activo sin validación previa** — en proyectos con modelos ML o sistemas en producción, cualquier cambio a código activo requiere validación OOS o equivalente antes de implementarse.
3. **Leer LESSONS.md antes de analizar** — si existe, contiene errores ya resueltos. No recomendar lo que ya fue descartado o corregido.
4. **Distinguir siempre entre:**
   - Código activo en producción → solo cambios con validación
   - Código de validación/sandbox → puede refactorizarse con cuidado
   - Código archivado/inválido → puede eliminarse, documentar por qué
5. **Sincronización training↔bot es deuda crítica** — en proyectos ML, cualquier divergencia entre fórmulas de entrenamiento y las del bot es deuda de máxima prioridad.

---

## Categorías de deuda técnica

| Tipo | Ejemplos | Riesgo |
|------|----------|--------|
| **Formula sync debt** | Fórmulas training ≠ bot, features desincronizadas | 🔴 Crítico — modelo inválido silencioso |
| **Code debt** | Lógica duplicada, abstracciones pobres, magic numbers | Bugs, desarrollo lento |
| **Architecture debt** | Módulos mal acoplados, responsabilidades mezcladas | Límites de escalado |
| **Test debt** | Sin tests unitarios, sin tests de integración, sin validación OOS | Regresiones silenciosas |
| **Dependency debt** | Librerías desactualizadas, dependencias sin mantenimiento | Vulnerabilidades de seguridad |
| **Documentation debt** | READMEs desactualizados, conocimiento tribal no documentado | Fricción en nuevas sesiones |
| **Dead code debt** | Archivos inválidos no eliminados, versiones obsoletas referenciadas | Confusión, errores de carga |
| **Infrastructure debt** | Deploys manuales, sin monitorización, configuración hardcodeada | Incidentes, recuperación lenta |

---

## Stack de referencia (Python ML/Trading)

Cuando el proyecto usa este stack, aplica estas reglas adicionales:

- **Python + scikit-learn**: verificar que `joblib` se usa para guardar/cargar modelos (no `pickle`)
- **ib_insync / IBKR**: los puertos de conexión deben estar en config, no hardcodeados
- **TimescaleDB / psycopg2**: las queries deben ir en `db_manager`, no dispersas en el bot
- **Modelos ML**: verificar que el archivo de entrenamiento y el bot usan exactamente las mismas fórmulas de features — cualquier divergencia es `formula sync debt` de prioridad máxima

---

## Framework de priorización

Puntúa cada ítem en:
- **Impacto**: ¿Cuánto ralentiza el desarrollo o causa errores? (1-5)
- **Riesgo**: ¿Qué pasa si no lo arreglamos? (1-5)
- **Esfuerzo**: ¿Qué tan difícil es el fix? (1-5, invertido — menor esfuerzo = mayor prioridad)
- **Riesgo bot activo**: ¿Toca código de producción activo? (sí/no)

**Prioridad = (Impacto + Riesgo) × (6 - Esfuerzo)**

---

## Output

| # | Tipo | Descripción | Impacto | Riesgo | Esfuerzo | Prioridad | Riesgo bot activo | Acción |
|---|------|-------------|---------|--------|----------|-----------|-------------------|--------|
| 1 | Formula sync | [descripción] | 5 | 5 | 2 | 40 | ✅ SÍ | Validar OOS antes |
| 2 | Dead code | [descripción] | 3 | 2 | 1 | 25 | ❌ NO | Eliminar |

Plan de remediación por fases:
- **Fase inmediata** (sin riesgo, < 1h): dead code, documentación, renombrados
- **Fase media** (validación necesaria, 1-4h): refactors de módulos no críticos
- **Fase cuidadosa** (requiere validación completa): cambios a bot activo, modelos, pipelines

---

## Al iniciar el análisis

```
1. Verificar si existe FROZEN_FILES.md → listar archivos protegidos
2. Leer LESSONS.md → registrar errores ya conocidos
3. Identificar qué es activo vs sandbox vs archivado
4. Ejecutar análisis por categorías
5. Presentar tabla priorizada + plan de remediación
```

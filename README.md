# Agile Delivery Team

Un equipo ágil de entrega impulsado por IA que convierte el output del Discovery Agent en un **plan de entrega listo para construir**: épicas priorizadas, backlog refinado, arquitectura con ADRs y un Sprint Plan verificado por gates de calidad.

---

## Qué hace este equipo

Toma las salidas de un Discovery Agent (personas, requisitos, user stories, MVP canvas, mapa de evidencia) y produce, en orden:

1. **Épicas y backlog** — el Product Owner descompone el MVP en épicas priorizadas por valor y genera el `backlog.json` estructurado.
2. **Historias refinadas** — el Developer refina cada historia candidata hasta que cumpla INVEST + Definition of Ready, con criterios de aceptación en Gherkin y estimación en puntos. Un gate automático (`dor-invest-gate.py`) bloquea la escritura si algo no cumple.
3. **Arquitectura y ADRs** — el Architect decide la estructura técnica mínima viable y registra el porqué de cada decisión en ADRs formato MADR.
4. **Sprint Plan** — el Scrum Master selecciona las historias Ready en orden de prioridad respetando la capacidad del equipo. El mismo gate lo custodia.
5. **Reporte visual** — un script determinista genera un dashboard HTML con el backlog por épica (INVEST a color) y el sprint plan.

---

## Los cuatro roles

| Rol | Pregunta de vida | Artefactos que produce |
|-----|-----------------|----------------------|
| **Product Owner** | ¿Estamos construyendo lo correcto? | `epics.md`, `backlog.json` |
| **Developer** | ¿Cómo lo construimos bien? | `stories.md`, `backlog.json` (actualizado) |
| **Scrum Master** | ¿Qué frena al equipo? | `sprint-plan.md`, `sprint-plan.json` |
| **Architect** | ¿Qué estructura sostiene esto en el tiempo? | `architecture.md`, `adr/ADR-NNNN-*.md` |

---

## Flujo de trabajo

```
# 1. Copia las salidas del Discovery Agent en inbox/
cp discovery-outputs/* deliveries/<nombre>/inbox/

# 2. Genera épicas y backlog
/delivery:generate-epics <nombre>

# 3. Refina historias (pasa por el gate DoR/INVEST)
/delivery:generate-stories <nombre>

# 4. Define la arquitectura y los ADRs
/delivery:architecture <nombre>

# 5. Arma el Sprint Plan (pasa por el gate)
/delivery:sprint-plan <nombre>

# 6. Genera el reporte visual HTML
/delivery:report <nombre>
```

---

## Estructura de un delivery

```
deliveries/<nombre>/
├── inbox/                        ← salidas del Discovery Agent (solo lectura)
│   ├── mvp-canvas.md
│   ├── user-stories.md
│   ├── requisitos.md
│   ├── personas.md
│   └── evidence-map.json
└── outputs/                      ← lo que produce este equipo
    ├── epics.md                  (Product Owner)
    ├── backlog.json              (PO + Developer — lo lee el gate)
    ├── stories.md                (Developer — GATED por DoR/INVEST)
    ├── architecture.md           (Architect)
    ├── adr/
    │   └── ADR-NNNN-<slug>.md   (Architect)
    ├── sprint-plan.md            (Scrum Master — GATED)
    ├── sprint-plan.json          (estructurado)
    └── report.html               (generado por scripts/build-report.py)
```

---

## Reglas no negociables

- **Cero invención.** Toda épica, historia o decisión traza al `inbox/`. Lo que no está respaldado por el discovery es un supuesto abierto (`open_questions`), no un hecho.
- **Trazabilidad.** Cada épica cita su origen en el MVP; cada historia cita su requisito o user story; cada ADR cita la fuerza que lo motiva.
- **INVEST + DoR es ley.** El hook `dor-invest-gate.py` bloquea la escritura de `stories.md` y `sprint-plan.md` si alguna historia no cumple. No se esquiva: se corrige.
- **Valor antes que solución.** Se prioriza por outcome → impacto, no por facilidad técnica.
- **Aislamiento.** Cada delivery vive en su carpeta. Los datos de un delivery nunca se mezclan con los de otro.

---

## Deliveries en este repositorio

| Delivery | Estado | Épicas | Historias | Sprint |
|----------|--------|--------|-----------|--------|
| [`dropshipping`](deliveries/dropshipping/) | Completo | 5 | 17 (55 pts) | Sprint 1: 7 historias · 20 pts |

---

## Requisitos

- Python 3.x (para el hook `dor-invest-gate.py` y el script `build-report.py`)
- Claude Code con acceso a los agentes en `.claude/agents/`


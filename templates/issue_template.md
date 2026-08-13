<!--
Plantilla de issue/epic — ver patrón 2 de METHODOLOGY.md.
Úsala desde el primer issue, no "cuando el equipo crezca".
Solo tiene sentido si hay más de un contribuidor (persona o agente IA)
trabajando en paralelo sin coordinarse en tiempo real; si trabajas solo,
un TODO simple ya es suficiente.
-->
---
name: <título descriptivo de la tarea>
status: open
created: <ISO 8601>
updated: <ISO 8601>
epic: "<nombre del epic, si aplica>"
priority: <P0|P1|P2|P3>
labels: ["<prioridad>", "tipo: feat|fix|research|chore", "<área>"]
github: <URL del issue, si existe>
depends_on: ["<epic/num o issue>"]
parallel: <true|false>
conflicts_with: []
---

# Task: <Título igual que `name`>

## Description

<Qué hay que hacer, por qué, y qué está mal/falta actualmente.>

## TDD Requirements

- Test RED: <qué test falla antes de implementar — debe poder escribirse
  antes de saber el detalle de la implementación>
- Test GREEN: <qué implementar para que pase>
- Refactor: <qué limpiar una vez pase, si algo>

## Acceptance Criteria

- [ ] <criterio verificable — con un número, un estado, o un comportamiento
      observable, no "que funcione bien">
- [ ] <criterio verificable>

## Constraints

- <restricciones técnicas o de negocio conocidas de antemano>
- **Depende de:** <issue/epic> | **Bloquea:** <issue/epic>

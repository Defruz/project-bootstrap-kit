# project-bootstrap-kit

Kit de arranque para proyectos nuevos. No es un framework ni una herramienta —
son plantillas de texto y una metodología escrita, para copiar al primer
commit de cualquier repo nuevo y adaptar a sus particularidades.

Origen: extraído y abstraído de la forma de trabajar en un proyecto real de
IA generativa + documentos + agentes (2026). Los ejemplos de `METHODOLOGY.md`
son genéricos a propósito — ningún nombre de cliente, proyecto cloud o
integración real aparece aquí.

## Cómo usarlo en un proyecto nuevo

1. Usa este repo como **template** (`gh repo create mi-proyecto --template Defruz/project-bootstrap-kit`
   o el botón "Use this template" en GitHub) o copia manualmente lo que necesites de `templates/`.
2. Lee `METHODOLOGY.md` una vez — son 10 patrones + un checklist + un criterio
   de "cuándo aplica esto y cuándo no".
3. Rellena `templates/CLAUDE.md.template` con los comandos y decisiones reales
   del proyecto nuevo. Empieza vacía la sección "Lecciones" — se rellena sola
   con el tiempo, según el patrón 1.
4. Adapta `templates/ci.yml.template` a tu stack real (los `<PLACEHOLDER>`
   marcan dónde).
5. Usa `templates/issue_template.md` para el primer issue, no para "cuando el
   proyecto crezca".
6. Descarta cualquier patrón marcado como "no aplica si..." que no encaje —
   no hay que forzarlos todos.

## Contenido

| Archivo | Qué es |
|---|---|
| `METHODOLOGY.md` | Los 10 patrones, con ejemplos abstraídos y el criterio de cuándo aplican |
| `templates/CLAUDE.md.template` | Esqueleto del doc de arranque, con la sección "Lecciones" explicada |
| `templates/ci.yml.template` | CI en capas (barato→caro, guardia anti-suite-vacía, suite lenta aislada) |
| `templates/pre-commit-config.yaml.template` | Espejo local del lint de CI |
| `templates/issue_template.md` | Plantilla de issue/epic con TDD Requirements + Acceptance Criteria |
| `templates/decision_log_template.md` | Formato para fijar un criterio de una frase ante una decisión recurrente |
| `templates/config_drift_check.py.template` | Esqueleto de guardia contra nombres de config que divergen en silencio |

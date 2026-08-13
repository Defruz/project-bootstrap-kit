<!--
Plantilla para fijar un criterio de una frase — ver patrón 8 de
METHODOLOGY.md. Se usa la primera vez que una decisión de diseño ambigua se
repite (la segunda vez que alguien duda lo mismo es la señal de que toca
escribir esto). Vive dentro de CLAUDE.md, en la sección "Decisiones de
arquitectura", o en un fichero aparte si son muchas.
-->

### ¿<Pregunta recurrente y ambigua del proyecto>? (decidido <AAAA-MM-DD>)

**Criterio (una frase):**

> "<La regla, formulada para que se pueda citar en una frase sin releer el
> contexto completo>"

**Ejemplos de aplicación:**

- <caso real> → <lado A del criterio>, porque <razón corta>.
- <caso real> → <lado B del criterio>, porque <razón corta>.

**Por qué se decidió así:** <el trade-off real que se sopesó, en 1-2 frases —
esto es lo que evita que alguien reabra el debate sin conocer la razón>.

**Se reabre si:** <condición concreta que invalidaría el criterio — p.ej. "el
coste de <X> baja de forma significativa" o "aparece un caso que no encaja en
ninguno de los dos lados"> .

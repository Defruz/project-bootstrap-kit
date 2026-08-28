# Metodología de arranque — 11 patrones

> Cada patrón separa la **regla** (se lleva siempre) del **ejemplo** (ilustra
> la regla con un caso genérico, no de un proyecto ni cliente concreto).
> Cuando un patrón no aplica a tu proyecto, se descarta — no hay que forzar
> los diez.

---

## 1. Documenta la razón, no solo la instrucción

**Regla:** cuando una corrección o confusión cuesta más de ~30 min de
diagnóstico, no termina en el fix — termina en una frase en el doc de arranque
del proyecto con: qué pasó, qué síntoma producía, y por qué no era obvio.

**Ejemplo genérico:** "No uses el entorno virtual de la carpeta `legacy/`
para levantar el servicio nuevo — resuelve a una versión de runtime distinta,
arranca sin quejarse, y el fallo real aparece mucho después como un error
502 que no menciona la causa. Ya costó una tarde entera de diagnóstico
equivocado."

**Por qué funciona:** una instrucción sin motivo se relee como regla
arbitraria y alguien la revierte en cuanto estorba. Con el coste explícito, se
vuelve auto-explicativa.

---

## 2. Issues/epics auto-contenidos: dependencias explícitas + TDD + criterios verificables

**Regla:** si hay más de un contribuidor (persona o agente IA) trabajando en
paralelo sin coordinarse en tiempo real, cada issue lleva: dependencias
declaradas (qué bloquea a qué), qué test falla antes de implementar y qué lo
hace pasar, y criterios de cierre en checkboxes verificables — no en prosa.

**Ejemplo genérico:** una tarea de "añadir caché a un endpoint" declara que
depende de la tarea que define el esquema de la clave de caché, lleva un test
que comprueba que la segunda llamada no repite el trabajo caro, y un criterio
de cierre medible ("p95 de la segunda llamada < 50ms"), no "que vaya rápido".

**No aplica si:** trabajas solo, sin agentes IA generando trabajo en paralelo
— ahí un TODO.md o el propio issue tracker sin plantilla ya es suficiente.

---

## 3. Los tests codifican lecciones operativas, no solo corrección

**Regla —** cuatro tipos de test más allá de "¿la función devuelve lo
esperado?":
1. **Test-guardia de entorno/infraestructura** — algo que puede fallar en
   silencio y aparecer después como un síntoma irreconocible se convierte en
   un test que falla rápido y explica el síntoma en el mensaje de error.
2. **Test de coherencia interna de datos de prueba** — no solo que el campo
   exista, sino que los datos del fixture sean internamente consistentes.
3. **Test exhaustivo sobre mapeos config→comportamiento** — parametrizar
   todos los casos conocidos, no una muestra, porque este tipo de código
   suele fallar en silencio (cae a un valor por defecto) en vez de romper.
4. **Vigilar los `skip`/`skipif` a nivel de módulo** — pueden colapsar N tests
   en un solo "skipped" y una suite que parece completa sin estarlo.

**Ejemplo genérico:** un test que arranca el servicio con una versión de
runtime antigua y comprueba que falla con un mensaje que menciona la versión
mínima requerida y el síntoma real (no solo "unsupported version"). Un test
que recorre cada valor de un enum de configuración y comprueba que resuelve al
handler correcto, no solo dos o tres valores de ejemplo.

---

## 4. CI en capas: barato y rápido antes de caro y lento

**Regla:** ordenar los jobs de CI de más barato/rápido a más caro/lento, con
un guardia explícito contra "la suite de tests desapareció y CI sigue en
verde", y las suites lentas o frágiles en un job aparte, no bloqueante hasta
que demuestren ser fiables.

**Ejemplo genérico:** `lint` (segundos) → `smoke test` de imports/arranque
básico (segundos) → suite unitaria (con guardia: falla si no encuentra
ningún fichero de test) → suite end-to-end con navegador o contenedor
(minutos, aislada en su propio job).

---

## 5. Pre-commit local que refleja el lint de CI

**Regla:** lo que el job de lint de CI comprueba, se comprueba también antes
de que exista el commit — para que CI confirme en vez de descubrir.

**Ejemplo genérico:** un hook de pre-commit que corre el mismo formatter y
linter que el job `lint`, para no acumular commits de seguimiento del tipo
"aplicar formato" que un hook habría evitado.

---

## 6. Guardia contra nombres de configuración que divergen en silencio

**Regla:** cualquier configuración por convención de nombre (variable de
entorno, flag, nombre de recurso) que tenga un **fallback silencioso** si no
se encuentra es un lugar donde un typo no produce error, produce
comportamiento distinto sin avisar. Un test o script en CI compara lo que el
código referencia contra lo que la documentación declara, y falla si diverge.

**Ejemplo genérico:** el código lee `SERVICE_A_API_KEY`, pero el `.env.example`
documenta `SERVICE_A_KEY` — nadie ve un error, el servicio simplemente cae a
un modo degradado más lento. Un script que extrae las claves de `os.environ`
(o equivalente) usadas en el código y las compara con las documentadas
detecta esto sin depender de que alguien relea la documentación.

**No aplica si:** el proyecto valida su configuración al arrancar y falla
explícitamente ante cualquier variable ausente o desconocida — ahí el propio
arranque ya hace de guardia.

---

## 7. Convención de commits/ramas, con re-verificación si algo puede romperla

**Regla:** elegir una convención desde el principio, y si algo en el entorno
(agente, plantilla, herramienta) puede generar nombres fuera de convención
automáticamente, el hábito es "compruébalo y corrígelo en el primer minuto, y
otra vez antes de cada commit" — no "no lo hagas", porque el mecanismo
automático puede romperlo de nuevo a mitad de sesión.

**Ejemplo genérico:** `tipo(ámbito): descripción` en los commits;
`feature/issue-<N>` en las ramas; si una herramienta de IA crea ramas con un
hash ilegible, renombrarla en cuanto se detecta, sin esperar a que alguien lo
señale.

---

## 8. Para cada decisión recurrente y ambigua, fija un criterio de una frase — una sola vez

**Regla:** todo proyecto acumula preguntas de diseño que se repiten y no
tienen respuesta obvia por tipo de dato (dependen de juicio). En vez de
discutirlas cada vez, se decide una vez, se escribe como regla de una frase
con 1-2 ejemplos de cada lado, y se cita esa regla en vez de reabrir el
debate.

**Ejemplo genérico:** "si la tarea se puede escribir como función
determinista y probarla con asserts, no necesita un agente IA; si requiere
interpretar lenguaje, sopesar criterios no reducibles a umbrales, o mantener
una conversación, sí." Otro proyecto puede no tener esta pregunta y en
cambio necesitar una para "¿esto vive en el cliente o en el servidor?" o
"¿este dato se cachea o se recalcula?" — lo que se copia es el hábito de
detectar cuál es la pregunta recurrente de *ese* proyecto y fijarla rápido,
no la pregunta en sí.

---

## 9. Dependencia externa no disponible: sustituto con el mismo contrato, marcado sin ocultarlo

**Regla:** cuando el proyecto depende de algo que no se puede conectar o
contratar todavía (API de terceros, servicio no habilitado, credencial
pendiente), se construye un sustituto que devuelve **la misma forma de
datos** que devolverá lo real, más un campo de procedencia explícito.
Cualquier vista que muestre ese dato lo señala de forma visible — nunca en
letra pequeña — porque una costura simulada sin señalar, en una demo, se lee
como funcionalidad real.

**Ejemplo genérico:** una integración de verificación de identidad de un
tercero, aún sin contrato firmado, se sustituye por una función que devuelve
`{resultado: ..., origen: "simulado", advertencia: "..."}`. Sustituir la
simulación por la llamada real es cambiar el cuerpo de la función, no el
contrato que consume el resto del sistema.

**No aplica si:** el proyecto no tiene dependencias externas pendientes de
contratar o conectar.

---

## 10. Cachea construcciones costosas de cliente/recurso

**Regla:** si algo con handshake o autenticación cara (cliente de SDK, pool
de conexión, modelo cargado en memoria) se crea dentro de la función que
atiende la petición, cada petición paga el arranque completo. Se convierte en
un singleton con un "calentamiento" explícito al arrancar el proceso. Al medir
latencia de cualquiera de estos, se descarta la primera ejecución — mide el
arranque, no la llamada.

**Ejemplo genérico:** un cliente de SDK cloud tarda 20 s en construirse
(autenticación, negociación TLS, descubrimiento de endpoint) frente a 1-2 s
de la llamada real. Si el cliente se crea por request, cada request paga los
20 s — la corrección es un singleton creado una vez al arrancar el proceso.

---

## 11. Limita la complejidad ciclomática — y déjala aplicar al propio LLM

**Regla:** el número de caminos independientes por función (ramas `if`/`for`/
`while`/`except`/`and`/`or`) se mide con una herramienta objetiva y se limita
con un umbral (10 es el valor de referencia habitual; hasta 15-20 si el
proyecto ya tiene deuda). No es solo higiene de mantenimiento: decirle
explícitamente a un LLM que su código tiene que pasar este lint hace que
genere funciones más simples desde el principio, en vez de depender de que
"sepa" no anidar demasiado.

**Cómo introducirlo sin romper CI de golpe (técnica de ratchet):** si el
proyecto ya tiene funciones complejas, activar el umbral estricto de golpe
rompe CI para código que nadie va a tocar hoy. En su lugar: se excluyen
explícitamente (`noqa`/comentario de supresión) las funciones que ya superan
el umbral, y se deja el umbral estricto activo para **todo lo nuevo**. Así el
gate bloquea complejidad nueva sin exigir refactorizar la deuda existente en
el mismo cambio que activa la regla.

**Ejemplo genérico por stack:**
- Python: `ruff` ya incluye el checker de McCabe — `select = [..., "C90"]` +
  `[tool.ruff.lint.mccabe] max-complexity = 10` en `pyproject.toml`. Cero
  dependencias nuevas si ya usas ruff.
- JS/TS: regla `complexity` de ESLint, o `eslint-plugin-sonarjs`.
- Go: `gocyclo` vía `golangci-lint`.
- Agnóstico de lenguaje: `lizard` como CLI standalone.

**Dónde encaja:** mismo sitio que el resto del lint — el job `lint` de CI
(patrón 4) y el pre-commit (patrón 5). No es una guía de comportamiento
(skill/prosa) — es un gate mecánico, igual que el resto del lint.

---

## Checklist portable

- [ ] Doc de arranque con comandos + sección de "por qué" para correcciones
      caras de diagnosticar (patrón 1).
- [ ] Plantilla de issue/epic con dependencias explícitas + TDD + Acceptance
      Criteria — solo si hay más de un contribuidor en paralelo (patrón 2).
- [ ] Los cuatro tipos de test del patrón 3, aplicados a lo que realmente
      pueda fallar en silencio en este proyecto.
- [ ] CI en capas barato→caro, con guardia anti-suite-vacía y suites
      lentas/frágiles aisladas (patrón 4).
- [ ] Pre-commit que refleje el lint de CI (patrón 5).
- [ ] Guardia contra divergencia de nombres de configuración, si aplica
      (patrón 6).
- [ ] Convención de commits/ramas + re-verificación (patrón 7).
- [ ] Identificar cuál es, en *este* proyecto, la decisión recurrente y
      ambigua que merece un criterio de una frase (patrón 8).
- [ ] Solo si hay dependencias externas no disponibles: sustituto con mismo
      contrato + procedencia visible (patrón 9).
- [ ] Cachear cualquier cliente/recurso con arranque caro, si lo hay
      (patrón 10).
- [ ] Gate de complejidad ciclomática en el lint (CI + pre-commit), con
      técnica de ratchet si hay deuda existente (patrón 11).

## Cuándo aplica todo esto (y cuándo no)

**Justifica el sistema completo cuando se cumplen dos o más:**
- El proyecto va a vivir semanas/meses.
- Hay más de un contribuidor (persona o agente) en paralelo sin coordinación
  en tiempo real.
- Se mezclan componentes no deterministas (LLMs/agentes) con lógica
  determinista.
- Hay dependencias externas no disponibles pero hace falta mostrar avance.
- Hay audiencia no técnica viendo demos o estado del proyecto.

**Usa algo más ligero (README + convención de commits + CI de una capa)
cuando:** es un script o prototipo de vida corta, un solo contribuidor, sin
componentes no deterministas en producción, sin demo a terceros.

**Regla práctica:** decídelo el primer día — retrofitear esto sobre un repo
que ya creció sin ello cuesta más que empezar con la versión completa cuando
ya se sabe que dos o más condiciones se van a cumplir.

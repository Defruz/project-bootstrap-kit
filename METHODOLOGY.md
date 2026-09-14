# Metodología de arranque — 12 patrones + extras

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

## 12. Gitignore por forma de nombre de credencial, no solo por `.env`

**Regla:** la mayoría de plantillas de `.gitignore` cubren `.env` y ya está —
pero la fuga típica en proyectos cloud no es `.env`, es una clave de service
account o un fichero de credenciales con un nombre que no sigue esa
convención. El `.gitignore` lleva patrones explícitos por **forma** de
nombre de credencial (no solo por extensión), y `.env.example` se fuerza a
quedar trackeado con una negación explícita, para que el contrato de
configuración documentado no desaparezca por accidente.

**Ejemplo genérico (evidencia real, patrones sin nombres de proyecto):**
```gitignore
# Env
.env
!.env.example

# Credenciales por forma de nombre — no solo por extensión
*.json.key
*-service-account*.json
service-account*.json
credentials*.json
application_default_credentials.json
```

**No aplica si:** el proyecto no usa credenciales basadas en fichero (todo
vía variables de entorno inyectadas por la plataforma, sin fichero de por
medio) — ahí basta con ignorar `.env`.

---

## 13. Higiene de git cuando varios agentes de IA trabajan en paralelo sobre worktrees

**Regla:** un flujo donde varios agentes de IA (o varias sesiones) trabajan en
paralelo, cada uno en su propio worktree y rama de git, introduce tres riesgos
que no existen con un único contribuidor secuencial:

1. **Limpieza automática de lo ya fusionado.** Cada worktree o rama que ya se
   mergeó (PR aceptado, rama remota borrada) se queda huérfano en local salvo
   que algo lo borre. Un hook de git (`post-merge`/`post-checkout`, activado
   solo al hacer pull de la rama principal) borra worktree y rama con
   `git worktree remove` (sin `--force`) y `git branch -d` (sin `-D`) — la
   propia orden se niega si hay cambios sin commitear o si la rama no está
   realmente mergeada, así que la seguridad la da git, no lógica propia de
   "¿está mergeada?".
2. **Sincronizar con el remoto antes de tocar nada, no solo antes de
   comitear.** Antes de editar o borrar contenido, comprobar si la copia
   local de la rama principal está por detrás del remoto
   (`git fetch` + `git rev-list --count HEAD..origin/main`) y sincronizar
   primero. La señal de alerta suele ser indirecta (un puerto de servidor ya
   ocupado, un conflicto al pushear) y para entonces ya se editó o borró
   contra una base desactualizada.
3. **No fiarse de la ruta de trabajo que reporta un agente.** Un subagente al
   que se le pide trabajar dentro de un worktree concreto puede resolver
   rutas relativas contra su propia idea de la raíz del repo en vez de contra
   el directorio que se le pasó, y escribir en el checkout equivocado sin dar
   ningún error — el fallo es silencioso. Pasarle siempre la ruta absoluta de
   salida, y verificar dónde escribió de verdad antes de continuar.

**Ejemplo genérico:** en un repo donde cada tarea de un agente de IA crea un
worktree con su propia rama, tras semanas de trabajo se acumulan decenas de
ramas locales con upstream ya borrado en remoto y varios worktrees enteros
apuntando a ramas ya fusionadas hace tiempo — nadie los había borrado a mano.
Al mismo tiempo, otro agente edita un módulo contra una copia local de la
rama principal varios commits por detrás, sin saber que de por medio se
fusionó una funcionalidad entera relacionada con ese módulo: si publica sin
sincronizar antes, pisa ese trabajo sin darse cuenta hasta el conflicto — o
sin conflicto siquiera, si el cambio no toca las mismas líneas.

**Por qué funciona:** los tres hábitos son baratos comparados con el coste de
perder trabajo ajeno o acumular basura que nadie limpia; y delegar la
comprobación de seguridad en los propios comandos de git (en vez de
reimplementar "¿está mergeada?" o "¿está sincronizada?") evita falsos
positivos.

**No aplica si:** un único contribuidor trabaja de forma secuencial, sin
worktrees ni ramas paralelas — ahí la copia local siempre refleja el estado
real y no hay nada que limpiar automáticamente.

---

## 14. La infraestructura declarativa gana siempre al dashboard — secretos en el gestor dedicado

**Regla:** si el despliegue de un servicio se define de forma declarativa en
un fichero versionado (Terraform, definiciones de Workers/Functions,
CloudFormation, Serverless Framework...), cualquier configuración puesta a
mano en el dashboard del proveedor que no esté reflejada en ese fichero se
pierde en el siguiente despliegue automático, sin aviso. Los valores
persistentes — sobre todo secretos — van en el gestor de secretos dedicado
del proveedor; el fichero declarativo guarda solo la referencia (nombre o
ID), nunca el valor.

**Ejemplo genérico:** alguien añade un token de autenticación a mano desde el
panel del proveedor para probar algo rápido. Funciona hasta el primer deploy
posterior: el fichero de infraestructura versionado sobrescribe la
configuración del servicio y el token desaparece sin que el deploy falle ni
avise. El error solo aparece después, como un fallo de autenticación que no
menciona la causa real. La corrección es mover el token al gestor de
secretos del proveedor y referenciarlo por nombre desde el fichero
declarativo, nunca pegar el valor directamente en él ni en el dashboard.

**Por qué funciona:** hace explícito en el propio repo cuál es la fuente de
verdad de la configuración, en vez de dejar que dashboard y código
declarativo compitan en silencio.

**No aplica si:** el despliegue se gestiona enteramente desde el dashboard,
sin fichero declarativo versionado — ahí no hay overwrite que temer (aunque
se pierde reproducibilidad, que es un problema aparte).

---

## 15. Lista blanca explícita para lo que se publica sin autenticación

**Regla:** cuando una carpeta o repo se publica tal cual a un sitio o CDN sin
autenticación delante, el fichero que decide qué se sube es una **lista
blanca** explícita de lo que debe ser público, no una lista negra de lo que
se excluye. Una lista negra falla en modo abierto: cualquier fichero nuevo
que alguien olvide añadir a las exclusiones se publica por defecto.

**Ejemplo genérico:** un sitio estático se despliega publicando el repo
entero salvo lo listado en un fichero de exclusiones. Meses después, alguien
añade una carpeta de notas internas o un fichero con una clave de prueba, y
se despliega igual — nadie recordó añadirlo a las exclusiones, y no hay
ningún error que lo avise, solo una fuga silenciosa. Invertir el fichero a
lista blanca (solo se publican estos directorios o ficheros, todo lo demás
queda fuera) hace que cualquier fichero nuevo quede fuera por defecto hasta
que alguien decida explícitamente hacerlo público.

**Por qué funciona:** falla cerrado en vez de abierto — un olvido produce
"esto no se ve" (se detecta rápido, porque falta algo esperado) en vez de
"esto se ve y no debería" (se detecta tarde, si es que se detecta).

**No aplica si:** el sitio está detrás de autenticación, o el propio
proveedor ya publica solo una carpeta de build explícita (p.ej. `dist/`) sin
posibilidad de incluir el resto del repo.

---

## Extras de estructura (no salen de hipotecas — son estándar, añadidos por completitud)

> A diferencia de los 12 patrones anteriores, esto no viene de observar un
> repo real con evidencia de que costó algo no tenerlo — es scaffolding
> convencional que cuesta poco y vale la pena tener desde el primer commit.

- **Plantilla de PR** (`.github/PULL_REQUEST_TEMPLATE.md`) — refuerza el
  patrón 2: si la plantilla trae un checkbox de "¿este PR cambia el estado
  del proyecto? ¿el doc de arranque está actualizado?", la documentación no
  llega como PR de seguimiento.
- **`.editorconfig`** — evita diffs de espacios/finales de línea entre
  editores distintos, sin depender de que cada contribuidor configure su
  editor a mano.
- **Actualización automática de dependencias** (Dependabot/Renovate) — para
  que las dependencias no se queden congeladas hasta que algo rompe por una
  CVE vieja.

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
- [ ] `.gitignore` con patrones por forma de nombre de credencial, no solo
      `.env` (patrón 12).
- [ ] Hook de limpieza de worktrees/ramas huérfanas + hábito de sincronizar
      antes de tocar archivos, si hay varios agentes de IA o sesiones en
      paralelo sobre el mismo repo (patrón 13).
- [ ] Secretos de despliegue en el gestor dedicado del proveedor, nunca como
      override manual sobre infraestructura declarativa (patrón 14).
- [ ] Lista blanca explícita de lo publicado, si el proyecto despliega a un
      sitio sin autenticación (patrón 15).
- [ ] Extras de estructura: plantilla de PR, `.editorconfig`, actualización
      automática de dependencias.

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

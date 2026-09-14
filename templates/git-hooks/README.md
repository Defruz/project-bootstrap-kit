# Hook de limpieza de worktrees/ramas — ver patrón 13 de METHODOLOGY.md

A diferencia del resto de `templates/`, esto **no se copia dentro de un repo
de proyecto** — es configuración de git a nivel de máquina, instalada **una
vez por equipo**, no una vez por proyecto. Resuelve un problema que aparece
en cualquier repo donde trabajes con varios agentes de IA o sesiones en
paralelo, cada uno en su worktree: worktrees y ramas que ya se fusionaron
(PR aceptado, rama remota borrada) se quedan huérfanos en local si nadie los
borra a mano.

## Instalación (una vez por máquina)

```bash
mkdir -p ~/.claude/git-hooks   # o cualquier ruta fuera de un repo de git
cp templates/git-hooks/post-merge.template ~/.claude/git-hooks/post-merge
cp templates/git-hooks/post-checkout.template ~/.claude/git-hooks/post-checkout
cp templates/git-hooks/limpiar-ramas-comun.sh.template ~/.claude/git-hooks/limpiar-ramas-comun.sh
chmod +x ~/.claude/git-hooks/post-merge ~/.claude/git-hooks/post-checkout ~/.claude/git-hooks/limpiar-ramas-comun.sh

git config --global core.hooksPath ~/.claude/git-hooks
```

`core.hooksPath` sustituye la búsqueda de hooks en **todos** tus repos por
esta carpeta compartida. Por eso `post-merge`/`post-checkout` delegan primero
a `$(git rev-parse --git-dir)/hooks/<hook>` o `.husky/<hook>` si existen, para
no romper hooks que ya tuvieras en un repo concreto (husky, pre-commit, etc.).

**Antes de instalarlo, comprueba que no tienes ya un `core.hooksPath` global**
(`git config --global --get core.hooksPath`) — si lo tienes, fusiona la
delegación en tu hook existente en vez de sobrescribirlo.

## Qué hace

Al hacer `git pull`/`merge` (o `checkout` de rama) estando en `main`/`master`
de cualquier repo: `git fetch --prune`, y para cada rama con upstream `[gone]`
(ya borrada en remoto tras mergear el PR): si tiene worktree asociado,
`git worktree remove` (sin `--force`) + `git branch -d` (sin `-D`); si no,
solo `git branch -d`. Sin lógica manual de "está mergeada" — la seguridad la
da que ambas órdenes se niegan solas si hay algo sucio o sin mergear.

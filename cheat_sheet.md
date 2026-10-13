# Git Cheat Sheet: Clase 1

## Los tres estados (modelo mental)

```
Directorio de trabajo  --git add-->  Staging  --git commit-->  Repo local  --git push-->  Remoto (GitHub)
                                                                    <--git pull / fetch--
```

---

## 0. Configuración inicial (una sola vez)

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@correo.com"
git config --global init.defaultBranch main
git config --list          # verificar la configuración
```

> Si no configuras nombre y correo, tus commits salen con autor equivocado.

---

## 1. Crear u obtener un repositorio

| Comando | Qué hace |
|---|---|
| `git init` | Convierte la carpeta actual en un repositorio nuevo (local). |
| `git clone <url>` | Descarga una copia completa de un repo remoto y la deja conectada a `origin`. |
| `git clone <url> <carpeta>` | Igual, pero en una carpeta con otro nombre. |
| `git remote add origin <url>` | Conecta un repo local (creado con `init`) con uno remoto. |
| `git remote -v` | Muestra los remotos configurados. |

**¿`init` o `clone`?** `init` empieza de cero; `clone` trae un proyecto que ya existe.

---

## 2. El ciclo diario: ver, preparar, guardar

| Comando | Qué hace |
|---|---|
| `git status` | Muestra qué archivos cambiaron, cuáles están en staging y en qué rama estás. **Úsalo todo el tiempo.** |
| `git diff` | Muestra los cambios que aún **no** están en staging. |
| `git diff --staged` | Muestra los cambios que **ya** están en staging (lo que irá en el commit). |
| `git add <archivo>` | Pasa un archivo a staging. |
| `git add .` | Pasa todos los cambios de la carpeta actual a staging. |
| `git commit -m "mensaje"` | Guarda en el repo local una "foto" de lo que está en staging. |
| `git log --oneline` | Lista los commits en una línea cada uno. |
| `git log --oneline --graph --all` | Igual, pero dibuja el grafo de ramas. |

```bash
git status
git add notas.txt
git commit -m "PROJ-12: agrega mi nombre a la lista"
```

**Buen mensaje de commit:** corto, en imperativo, explica qué y por qué. Incluye la clave de Jira.

---

## 3. Sincronizar con el remoto

| Comando | Qué hace |
|---|---|
| `git push` | Sube tus commits locales al remoto (la rama ya debe tener upstream). |
| `git push --set-upstream origin <rama>` | Sube una rama **nueva** por primera vez y la enlaza con su rama remota. Atajo: `git push -u origin <rama>`. |
| `git pull` | Trae los cambios del remoto y los integra en tu rama (`fetch` + `merge`). |
| `git fetch` | Solo descarga los cambios del remoto, sin integrarlos. |

```bash
# Primera vez que subes una rama nueva
git push --set-upstream origin feature/PROJ-12-mi-nombre

# Después, basta con
git push
```

**Si el push es rechazado:** el remoto tiene commits que tú no. Haz `git pull` y vuelve a intentar.

---

## 4. Ramas

| Comando | Qué hace |
|---|---|
| `git branch` | Lista las ramas locales (la actual lleva `*`). |
| `git branch -a` | Lista también las ramas remotas. |
| `git branch <nombre>` | Crea una rama (sin cambiarte a ella). |
| `git switch <rama>` | Cambia a una rama existente. |
| `git switch -c <rama>` | Crea una rama nueva **y** se cambia a ella. |
| `git checkout <rama>` | Equivalente antiguo de `switch`. |
| `git checkout -b <rama>` | Equivalente antiguo de `switch -c`. |
| `git branch -d <rama>` | Borra una rama local ya integrada. |

```bash
git switch dev
git pull
git switch -c feature/PROJ-12-mi-nombre
```

**Convención de nombres:** `feature/PROJ-12-descripcion`, `fix/PROJ-30-descripcion`.

---

## 5. Integrar ramas

Siempre te paras en la rama **destino** (la que recibe) y traes la otra.

### `git merge` (merge commit)

```bash
git switch main
git merge feature/PROJ-12-mi-nombre
```

- Une las ramas con un commit extra de "merge".
- Conserva todos los commits originales.
- Historial **no lineal**.

### `git merge --squash`

```bash
git switch main
git merge --squash feature/PROJ-12-mi-nombre
git commit -m "PROJ-12: agrega login"
```

- Junta todos los cambios de la rama en staging, **sin hacer el commit**: tú lo haces después.
- Resultado: **un solo commit** en `main`.
- Historial lineal. Ideal cuando los commits de la rama son desordenados.

### `git rebase`

```bash
git switch feature/PROJ-12-mi-nombre
git rebase main
```

- Reaplica los commits de tu rama **encima** de `main`, uno por uno.
- Historial **lineal** y conserva cada commit, pero con **hashes nuevos**.
- Como reescribe historia, si ya subiste la rama necesitas:

```bash
git push --force-with-lease
```

> **Regla de oro:** nunca hagas rebase de una rama que otras personas ya descargaron, y nunca fuerces push a `main` o `dev`.

### Comparación rápida

| | Merge | Squash | Rebase |
|---|---|---|---|
| Commits de la rama | Se conservan | Se aplastan en 1 | Se conservan (hashes nuevos) |
| Commit extra de merge | Sí | No | No |
| Historial | No lineal | Lineal | Lineal |
| Úsalo cuando | Importa la trazabilidad | Commits desordenados | Commits limpios y quieres historial lineal |

En GitHub, estas tres opciones aparecen en el botón verde del PR: **Create a merge commit**, **Squash and merge**, **Rebase and merge**.

---

## 6. Conflictos de merge

Un conflicto ocurre cuando dos ramas modificaron **la misma línea**. Git marca el archivo así:

```
<<<<<<< HEAD
Ana
=======
Luis
>>>>>>> feature/PROJ-13-luis
```

- Entre `<<<<<<<` y `=======`: **tu versión** (rama actual).
- Entre `=======` y `>>>>>>>`: **la versión que llega**.

**Cómo resolverlo:**

1. Edita el archivo y deja el resultado final (casi siempre, conservar **ambos** nombres).
2. **Borra** las tres líneas de marcadores (`<<<<<<<`, `=======`, `>>>>>>>`).
3. Guarda y cierra el conflicto:

```bash
git add notas.txt
git commit -m "Resuelve conflicto en notas.txt"
git push
```

**Traer los cambios de `dev` a tu rama para resolver en local:**

```bash
git switch feature/PROJ-12-mi-nombre
git pull origin dev
# resolver conflictos, y luego add + commit + push
```

> En VS Code aparecen los botones **Accept Current / Accept Incoming / Accept Both**.

---

## 7. Pull Request (PR)

Un **Pull Request** es una solicitud para integrar los cambios de una rama en otra (por ejemplo, de `feature/PROJ-12-mi-nombre` a `dev`). No es un comando de Git, sino una función de GitHub.

Un PR tiene:

- **Rama origen** (*compare*): la que contiene tus cambios.
- **Rama destino** (*base*): la que los va a recibir.
- **Título y descripción**: qué cambia y por qué (incluye la clave de Jira).
- **Revisión de código**: otros miembros del equipo comentan, piden cambios o aprueban.
- **Verificación de conflictos**: GitHub avisa si la rama no se puede integrar limpiamente.
- **Botón de merge**: al aprobarse, se elige la estrategia (merge, squash o rebase).

**¿Para qué sirve?** Para que ningún cambio entre a una rama compartida sin que alguien más lo haya revisado.

---

## 8. `.gitignore`

Archivo en la raíz del repo con lo que Git **no** debe rastrear:

```
node_modules/
.env
*.log
.vscode/
__pycache__/
```

> Si un archivo ya fue commiteado, agregarlo al `.gitignore` no basta: hay que quitarlo del repo con `git rm --cached <archivo>`.
> Nunca subas contraseñas ni llaves API.

---

## Flujo completo en un vistazo

```bash
git clone <url>
cd <repo>
git switch dev
git pull
git switch -c feature/PROJ-12-mi-nombre
# ...editar archivos...
git status
git add .
git commit -m "PROJ-12: descripción del cambio"
git push --set-upstream origin feature/PROJ-12-mi-nombre
# abrir Pull Request hacia dev en GitHub
```
# Git en pareja: de `main` a las pull requests

Vais a mantener una pequeña web entre dos personas. Ya conocéis comandos de Git; ahora los usaréis con repositorios reales, un remoto compartido y cambios de otra persona.

**El objetivo no es evitar todos los conflictos, sino entender cuándo aparecen y resolverlos sin perder trabajo.**

## Cómo trabajar

- Formad una pareja y repartid los roles **A** y **B**. Ambos programan, integran cambios y revisan.
- Cada persona usa su propia máquina virtual Debian, su cuenta de GitHub y su repositorio local. Compartís **un único repositorio en GitHub**, propiedad de A, con B como colaborador.
- Seguid el orden. **🛑 Sincronización** significa que nadie continúa hasta que ambos hayáis alcanzado ese punto.
- Cuando una tarea indique «solo A» o «solo B», la otra persona espera. No hagáis `pull` por costumbre: en algunos pasos necesitamos conservar una copia desactualizada.
- Los bloques de comandos se ejecutan dentro del repositorio local, salvo indicación contraria. Sustituid los nombres y las rutas de ejemplo por los vuestros.
- En esta práctica no uséis `push --force`, `reset --hard`, rebase ni el botón **Sync Fork**. No vamos a trabajar con forks.
- Guardad las capturas fuera de la carpeta del repositorio hasta el apartado de entrega. No incluyáis contraseñas, tokens ni datos personales innecesarios.

### Recorrido

| Nivel | Trabajo | Qué vais a observar |
| --- | --- | --- |
| 1. `main` compartida | Commits, push, fetch, merge y pull | Un push rechazado sin conflicto y otro con conflicto |
| 2. Ramas | Una mejora por rama e integración desde terminal | Separar el trabajo e integrar con orden |
| 3. Pull requests | Revisión cruzada e integración desde GitHub | Una PR que necesita resolver un conflicto |

La entrega incluye **enlace al repositorio, capturas y respuestas razonadas**. Consultad los requisitos finales antes de empezar.

## La web inicial

- `index.html`: título, descripción y botón.
- `style.css`: una tarjeta y sus estilos.
- `script.js`: al pulsar el botón, aparece un saludo.

No necesita instalar librerías ni herramientas de construcción. Abrid `index.html` con el navegador; por ejemplo, desde un escritorio Linux:

```bash
xdg-open index.html
```

Recargad el navegador después de modificar archivos. Comprobar que Git ha integrado un cambio **no sustituye** comprobar que la web funciona.

---

# Preparación: un remoto, dos repositorios locales

## 0.0. Configuración previa: proxy de red (opcional)

Si la red de vuestro centro educativo requiere el uso de un servidor proxy para navegar o descargar paquetes, debéis configurarlo en el sistema para que `git`, `gh`, el navegador y `apt` tengan salida a Internet.

En vuestra MV Debian, podéis declarar las variables de entorno globales editando el archivo `/etc/environment` como administrador:

```bash
sudo nano /etc/environment
```

Añadid las siguientes líneas al final del archivo (sustituyendo la dirección si el docente indica otra):

```ini
http_proxy="http://192.168.0.11:3128"
https_proxy="http://192.168.0.11:3128"
HTTP_PROXY="http://192.168.0.11:3128"
HTTPS_PROXY="http://192.168.0.11:3128"
no_proxy="localhost,127.0.0.1"
NO_PROXY="localhost,127.0.0.1"
```

Guardad los cambios (`Ctrl + O`, `Enter` y `Ctrl + X`) y aplicadlos a vuestra sesión de terminal actual:

```bash
source /etc/environment
export http_proxy https_proxy HTTP_PROXY HTTPS_PROXY no_proxy NO_PROXY
```

> **Consejo para APT:** En sistemas Debian, `sudo` puede limpiar las variables de entorno por defecto. Si `sudo apt update` da errores de conexión o tiempo de espera, configurad el proxy para APT directamente:
> ```bash
> echo 'Acquire::http::Proxy "http://192.168.0.11:3128/";' | sudo tee /etc/apt/apt.conf.d/99proxy
> echo 'Acquire::https::Proxy "http://192.168.0.11:3128/";' | sudo tee -a /etc/apt/apt.conf.d/99proxy
> ```

## 0.1. Revisar las herramientas — ambos

```bash
git --version
gh --version
```

Si Git no está instalado, en vuestra MV Debian y con autorización para administrar el sistema:

```bash
sudo apt update
sudo apt install git
```

Si `gh` no está instalado en el sistema, lo instalaréis en el apartado 0.3.

Comprobad también que podéis iniciar sesión en GitHub desde el navegador.

> **Píldora: identidad y autenticación son cosas distintas.**
> `user.name` y `user.email` identifican la autoría de los commits. No sirven para iniciar sesión en GitHub. Usad un correo asociado a vuestra cuenta o el correo privado `noreply` que GitHub muestra en **Settings → Emails**. Los commits publicados pueden exponer ese correo.

## 0.2. Crear el repositorio compartido — solo A

1. En GitHub, pulsad **New repository**.
2. Elegid vuestra cuenta como propietaria y el nombre `practica-git-pareja`.
3. Elegid la visibilidad que indique el docente. Si es privado, tendréis que darle acceso para corregirlo.
4. **No** inicialicéis README, `.gitignore` ni licencia: queremos un repositorio remoto vacío.
5. Creadlo y copiad su URL HTTPS, con esta forma:
   `https://github.com/USUARIO_A/practica-git-pareja.git`.
6. En **Settings → Collaborators** (el nombre puede variar), invitad a la cuenta de B.

**B:** aceptad la invitación desde GitHub o el correo y comprobad que podéis acceder al repositorio. No creéis otro repositorio ni hagáis un fork.

## 0.3. Autenticarse con GitHub — ambos

Los comandos de esta práctica usan HTTPS. GitHub **no acepta la contraseña de la cuenta** como contraseña para hacer operaciones Git por HTTPS.

La forma oficial y más cómoda de autenticarse en terminal es mediante **GitHub CLI (`gh`)**, que además configura Git automáticamente para recordar vuestra sesión sin pedir usuario ni contraseña en cada operación.

### 1. Instalar GitHub CLI (`gh`)

Comprobad si ya lo tenéis instalado:

```bash
gh --version
```

Si no está instalado, en vuestra MV Debian añadid el repositorio oficial e instaladlo:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings \
&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
&& cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

### 2. Iniciar sesión y vincular Git

Iniciad sesión ejecutando:

```bash
gh auth login
```

Responded a las preguntas interactivas con estas opciones:

1. **What account do you want to log into?** → Seleccionad `GitHub.com`.
2. **What is your preferred protocol for Git operations on this host?** → Seleccionad `HTTPS`.
3. **Authenticate Git with your GitHub credentials?** → Elegid `Yes` *(fundamental: esto configura el asistente de credenciales de Git para que clone, push y pull no vuelvan a pedir contraseña)*.
4. **How would you like to authenticate GitHub CLI?** → Elegid `Login with a web browser`.

El terminal mostrará un código de 8 caracteres (tipo `XXXX-XXXX`). Pulsad `Enter` para abrir el navegador (o abrid `https://github.com/login/device` si el navegador no se abre solo) e introducid ese código para autorizar el equipo.

### 3. Comprobar la conexión

Confirmad que la sesión está activa y asociada a vuestro usuario:

```bash
gh auth status
```

Debe mostrar vuestro nombre de usuario y que las operaciones Git están configuradas por HTTPS.

> **Alternativa SSH:** Si ya disponíais de SSH previamente configurado y vinculado a vuestra cuenta de GitHub, podéis usarlo sustituyendo la URL HTTPS por la URL SSH que ofrece GitHub. Para la práctica, la vía con `gh` por HTTPS es la opción recomendada.

## 0.4. Crear el primer historial — solo A

Descargad o descomprimid el material inicial que facilite el docente **fuera** de la futura carpeta de trabajo. Necesitáis únicamente `index.html`, `style.css` y `script.js`.

Desde vuestra carpeta de proyectos:

```bash
mkdir practica-git-pareja
cd practica-git-pareja
git init -b main
git config user.name "Nombre de A"
git config user.email "CORREO_DE_A"
```

Copiad los tres archivos a esta carpeta con el gestor de archivos. No copiéis una carpeta `.git` ni inicialicéis otro repositorio dentro. También podéis usar terminal, sustituyendo la ruta de ejemplo por la del material descomprimido:

```bash
cp /ruta/al/material/index.html /ruta/al/material/style.css /ruta/al/material/script.js .
```

Revisad y publicad el primer commit:

```bash
git status
git add index.html style.css script.js
git diff --staged
git commit -m "Crear la web inicial"
git remote add origin https://github.com/USUARIO_A/practica-git-pareja.git
git push -u origin main
```

> **Píldora: las tres zonas.**
> Editar modifica el directorio de trabajo. `git add` prepara una versión concreta en el área de preparación (*staging*). `git commit` guarda esa versión en el historial **local**. `git push` publica commits en el remoto. Guardar un archivo no hace ninguna de esas tres últimas cosas.
>
> `origin` es un nombre local para la URL del remoto. `-u` establece la relación de seguimiento entre la rama local y la remota; después suele bastar `git push`.

## 0.5. Clonar el trabajo — solo B

Desde vuestra carpeta de proyectos, no desde otro repositorio:

```bash
git clone https://github.com/USUARIO_A/practica-git-pareja.git
cd practica-git-pareja
git config user.name "Nombre de B"
git config user.email "CORREO_DE_B"
```

> **Píldora: clonar no es descargar un ZIP.**
> `clone` copia archivos e historial, configura `origin` y crea una rama local que sigue a la remota. Un ZIP no incluye esa relación ni el historial de Git.

## 0.6. Comprobar el punto de partida — ambos

Configurad este repositorio para usar merges al hacer pull. La configuración es local al proyecto, no global:

```bash
git config pull.rebase false
git config pull.ff true
git config merge.conflictStyle merge
git status
git remote -v
git branch -vv
git log --oneline --decorate --all
```

`pull.ff true` permite avanzar directamente cuando sea posible y permite crear un merge cuando las historias divergen. `merge.conflictStyle merge` hará que ambos veáis las mismas marcas básicas de conflicto.

Abrid la web y pulsad el botón.

**🛑 Sincronización 0:** ambos estáis en `main`, con el mismo commit inicial, sin cambios pendientes, con acceso al mismo remoto y con la web funcionando.

---

# Nivel 1. Colaborar directamente en `main`

## 1.1. Cambios independientes y push rechazado

> **Píldora: tener permiso no significa poder subir cualquier historial.**
> Un push normal no sobrescribe commits ajenos. Si la rama remota contiene commits que vuestra rama no incorpora, Git suele rechazarlo como *non-fast-forward* o *fetch first*. Eso no significa necesariamente que haya un conflicto de contenido.

**Primero, ambos trabajáis sin hacer pull:**

- **A:** en `index.html`, cambiad únicamente el texto del párrafo `descripcion` por `Una web creada en equipo para practicar Git y GitHub.`.
- **B:** en `style.css`, cambiad únicamente el fondo de `body`, de `#f1f5f9` a `#eef2ff`.

Cada persona revisa y confirma **su propio archivo**:

```bash
git diff
# A ejecuta:
git add index.html
git commit -m "Actualizar la descripción del encuentro"
```

```bash
# B ejecuta:
git add style.css
git commit -m "Cambiar el fondo de la página"
```

**🛑 Sincronización 1:** ambos tenéis un commit local nuevo, pero ninguno lo ha subido.

1. **Solo B:** ejecutad `git push origin main` y confirmad que termina correctamente.
2. **Solo A:** sin hacer pull ni fetch antes, ejecutad `git push origin main`.
3. **A:** guardad una captura del rechazo. No intentéis forzarlo.

**Antes de continuar, A predice y B contrasta:** ¿qué referencia actualizará `fetch`? ¿Cambiará los archivos de trabajo? ¿Esperáis que el merge necesite una resolución manual? Responded oralmente y contrastadlo con el resultado; no hace falta entregar estas predicciones.

**A: investigad e integrad por separado:**

```bash
git fetch origin
git log --oneline --graph --decorate --all
git merge origin/main -m "Integrar la descripción y el fondo"
git push origin main
```

El merge debería terminar **sin conflicto**: habéis cambiado archivos distintos.

**B:** incorporad el resultado:

```bash
git pull --no-rebase origin main
```

**Ambos:** comprobad la descripción y el fondo en el navegador y ejecutad:

```bash
git status
git log --oneline --graph --decorate --all
```

> **Píldora: `fetch`, `merge` y `pull`.**
> `fetch` actualiza vuestra información sobre el remoto, pero no integra sus cambios en la rama actual. `origin/main` es vuestra última referencia conocida de `main` en el remoto, no una consulta en directo a GitHub.
>
> `merge origin/main` integra esa referencia en la rama actual. `pull --no-rebase` combina descarga e integración mediante merge. Puede hacer un avance directo (*fast-forward*), crear un commit de merge o detenerse por conflictos.
>
> Antes de hacer fetch, `status` puede no avisar de commits recién subidos por otra persona: todavía no los conoce.

**Pregunta 1:** ¿por qué se rechazó el push de A si no habíais editado el mismo archivo? ¿Qué hicieron `fetch`, `merge` y `push`? ¿Qué permitió observar `log`?

**🛑 Sincronización 2:** ambos tenéis el mismo `main`, con los dos cambios, sin trabajo pendiente.

## 1.2. Un conflicto intencionado

Ahora editaréis exactamente la misma línea de `style.css`. **No hagáis pull entre la edición local y el intento de push de B.**

- **A:** cambiad `--color-principal: #2563eb;` por `--color-principal: #7c3aed;`.
- **B:** cambiad la misma línea original por `--color-principal: #059669;`.

Cada persona ejecuta:

```bash
git diff
git add style.css
git commit -m "Proponer un nuevo color principal"
```

**🛑 Sincronización 3:** ambos habéis confirmado vuestro color, ninguno lo ha subido aún.

1. **A:** `git push origin main`. Esperad a que termine.
2. **B:** `git push origin main`. Debe rechazarse.
3. **Antes de integrar, ambos predecís:** ¿el rechazo del push demuestra que hay un conflicto de contenido? ¿Qué esperáis que ocurra ahora al hacer pull y por qué? Contrastadlo después con lo observado.
4. **B:** integrad lo que falta:

```bash
git pull --no-rebase origin main
```

Esta vez Git debe detenerse con un conflicto en `style.css`.

**B: antes de resolverlo**, ejecutad `git status`, abrid el archivo y guardad una captura de las marcas:

```css
<<<<<<< HEAD
  --color-principal: #059669;
=======
  --color-principal: #7c3aed;
>>>>>>> identificador-del-commit
```

La etiqueta final puede ser distinta. En este merge, la parte superior es la versión local de B y la inferior es la versión que se está incorporando. No memoricéis «arriba = mi compañero»: depende de la operación y de dónde la ejecutáis.

> **Píldora: Git no decide el diseño.**
> Un conflicto significa que Git no puede combinar automáticamente ciertas modificaciones. Hay que acordar el resultado, editar el archivo, eliminar las marcas y confirmar la resolución. No basta con borrar las marcas si el contenido restante no tiene sentido.

**Ambos:** acordad un tercer color para conservarlo como decisión conjunta: `#b45309`.

**Solo B:** dejad esa línea así, sin ninguna marca ni colores duplicados:

```css
  --color-principal: #b45309;
```

Revisad y terminad el merge:

```bash
git diff
git diff --check
git add style.css
git diff --staged
git status
git commit -m "Resolver el conflicto de color con una propuesta común"
git push origin main
```

`git add` indica a Git que habéis resuelto el archivo. El commit concluye el merge pendiente; no es necesario empezar otro pull.

**A:** ejecutad `git pull --no-rebase origin main`.

**Ambos:** abrid la web y comprobad título y botón. Ejecutad `git status` y comparad la salida de `git rev-parse HEAD`: debe coincidir.

**Pregunta 2:** ¿en qué se diferencia este caso del anterior? ¿En qué momento apareció el conflicto: al hacer push o al integrar? ¿Qué papel tuvieron `add` y `commit` al resolverlo?

**🛑 Sincronización 4:** mismo commit, color acordado, web funcional y directorios de trabajo limpios.

---

# Nivel 2. Trabajar en ramas e integrar desde terminal

## 2.1. Una mejora por rama

> **Píldora: una rama no es otra carpeta.**
> Una rama es una referencia a una línea de trabajo. `git switch -c nombre` crea una rama desde el commit actual y cambia a ella. Los commits nuevos avanzan esa rama, no `main`. Antes de cambiar de rama, comprobad que no dejáis trabajo pendiente.

Partid ambos del `main` sincronizado del nivel anterior.

**A:**

```bash
git switch -c mejora/pie
```

En `index.html`, añadid justo antes de `</main>`:

```html
    <footer>Hecho en equipo con Git.</footer>
```

```bash
git diff
git add index.html
git commit -m "Añadir un pie a la tarjeta"
git push -u origin mejora/pie
```

**B:**

```bash
git switch -c mejora/movil
```

Añadid al final de `style.css`:

```css
@media (max-width: 600px) {
  body {
    padding: 1rem;
  }

  .tarjeta {
    margin: 0;
    padding: 1rem;
  }
}
```

```bash
git diff
git add style.css
git commit -m "Adaptar la tarjeta a pantallas pequeñas"
git push -u origin mejora/movil
```

**Ambos:** comprobad en GitHub que existen las ramas y que `main` todavía no contiene las mejoras. **No creéis una pull request aún.**

**🛑 Sincronización 5:** ambas ramas están publicadas y `main` sigue igual.

## 2.2. Integrar las dos mejoras, en orden

> **Píldora: importa en qué rama estáis.**
> `git merge otra-rama` incorpora `otra-rama` a la rama **actual**. `--no-ff` conserva un commit de merge aunque pudiera hacerse un avance directo; aquí lo usamos para reconocer las integraciones en el gráfico.

**Antes de ejecutar, B predice y A contrasta:** ¿qué rama cambiará con `git merge --no-ff mejora/movil`? ¿Por qué aparecerá un commit de merge aunque `main` pudiera avanzar directamente? Comprobad después vuestra predicción en el gráfico.

**Primero B**, mientras A espera:

```bash
git switch main
git pull --no-rebase origin main
git merge --no-ff mejora/movil -m "Integrar la mejora para móviles"
git push origin main
```

**Después A**, cuando B confirme que ha subido:

```bash
git switch main
git pull --no-rebase origin main
git merge --no-ff mejora/pie -m "Integrar el pie de la tarjeta"
git push origin main
```

**Por último B:**

```bash
git pull --no-rebase origin main
```

**Ambos:**

```bash
git fetch origin
git branch -a
git log --oneline --graph --decorate --all
git status
```

Probad la web y reducid el ancho del navegador. Guardad una captura del gráfico con las dos integraciones.

**Pregunta 3:** ¿por qué subir una rama no cambió `main`? ¿A qué rama afectó cada merge? ¿Para qué sincronizó A su `main` antes de integrar el pie?

**🛑 Sincronización 6:** ambos estáis en el mismo `main`, con pie y adaptación móvil, sin cambios pendientes. Conservad las ramas por ahora para facilitar la inspección.

---

# Nivel 3. Pull requests y revisión cruzada

Ahora nadie subirá directamente a `main` para incorporar las mejoras de este nivel. Publicaréis ramas y las integraréis con pull requests.

## Antes de empezar: ¿qué es una pull request?

Hasta ahora habéis integrado con `git merge` en terminal y publicado el resultado. Una **pull request (PR)** permite proponer esa integración en GitHub y conversar sobre ella **antes de incorporarla a la rama compartida**. No necesitáis haber hecho ninguna: vais a practicar el proceso completo.

Por ejemplo: «He cambiado el saludo en mi rama. Propongo incorporarlo a `main`; ¿puedes revisar el código y probarlo primero?». La PR es la propuesta y el espacio de revisión, **no otra copia de los archivos ni un comando de Git**. Tampoco es lo mismo que `git pull`, que descarga e integra cambios en vuestro repositorio local.

### Las partes de una PR

- **Base:** rama que recibirá los cambios; aquí, `main`.
- **Compare:** rama que propone los cambios; por ejemplo, `propuesta/saludo-b`. La dirección es `propuesta/saludo-b` → `main`, no al revés.
- **Título y descripción:** explican qué cambia, por qué y cómo se ha probado.
- **Conversation:** conversación, revisiones y estado de la propuesta.
- **Commits:** historial de commits de la propuesta.
- **Files changed:** diferencias de código (*diff*) que hay que revisar.

### Del cambio a la integración

1. La persona autora crea una rama, modifica, prueba, hace commit y **publica la rama con push**. Esto no cambia `main`.
2. Abre una PR con esa rama como compare y `main` como base. **Abrirla tampoco cambia `main`.** No hace falta un fork: las dos ramas están en vuestro repositorio compartido.
3. La otra persona lee el diff, prueba la propuesta y envía una revisión. Puede comentar (**Comment**), pedir correcciones (**Request changes**) o aprobar (**Approve**). Aprobar expresa conformidad; **no integra los cambios**.
4. Si hay correcciones, la persona autora hace nuevos commits y los sube a **la misma rama**. La PR se actualiza automáticamente y hay que revisar el resultado actualizado.
5. Cuando esté lista, alguien con permisos pulsa **Merge pull request** y confirma la integración. En esta práctica elegiréis **Create a merge commit**. Solo entonces se incorpora la propuesta a `main` en GitHub; los repositorios locales aún deben actualizarse con pull.

GitHub puede indicar que una PR se puede integrar sin conflictos, pero eso **no significa que esté revisada ni que funcione**. Si indica un conflicto, primero habrá que resolverlo; lo practicaréis con la segunda PR. **Cerrar una PR sin integrarla (Close)** descarta esa propuesta de integración sin incorporar sus cambios a `main`.

## 3.1. Dos propuestas desde la misma base

**A**, desde `main` sincronizado:

```bash
git switch -c propuesta/saludo-a
```

Cambiad solo la asignación del saludo en `script.js` por:

```js
  mensaje.textContent = "¡Hola! Esta web también se construye revisando en equipo.";
```

```bash
git diff
git add script.js
git commit -m "Proponer un saludo sobre la revisión en equipo"
git push -u origin propuesta/saludo-a
```

**B**, también desde el mismo `main` sincronizado y **antes de integrar ninguna PR**:

```bash
git switch -c propuesta/saludo-b
```

Cambiad la misma asignación original por:

```js
  mensaje.textContent = "¡Bienvenidas y bienvenidos al encuentro de clase!";
```

```bash
git diff
git add script.js
git commit -m "Proponer un saludo de bienvenida"
git push -u origin propuesta/saludo-b
```

**Cada persona** abre una PR en **Pull requests → New pull request**:

- `base: main`.
- `compare: propuesta/saludo-a` o `propuesta/saludo-b`, según corresponda.
- Comprobad la dirección de integración y revisad el diff antes de continuar.
- Título descriptivo; por ejemplo, `Proponer un saludo de bienvenida`.
- Descripción con **qué cambia, por qué y cómo se ha probado**. Indicad la acción de prueba y el resultado observado, no solo «funciona».
- Pulsad **Create pull request**; no la dejéis como borrador (*Draft*).
- Solicitad revisión a la otra persona si GitHub ofrece el selector; si no, avisadle compartiendo el enlace.

**🛑 Sincronización 7:** las dos PR están abiertas, ambas ramas parten de la misma base y ninguna se ha integrado todavía. En este momento pueden figurar ambas como integrables: el conflicto aparecerá después de integrar la primera.

## 3.2. Revisar e integrar la PR de B — A revisa

**A:** abrid la PR de B y revisad **Files changed**. No os limitéis a pulsar aprobar.

Para probar exactamente esa propuesta en vuestra MV, sin mezclarla con la vuestra y con el directorio de trabajo limpio:

```bash
git fetch origin
git switch --detach origin/propuesta/saludo-b
```

Abrid la web, recargad y pulsad el botón. No hagáis commits en este estado. Después, volved a vuestra rama:

```bash
git switch propuesta/saludo-a
```

> **Píldora: inspeccionar un commit.**
> `--detach` permite visitar un commit sin situarse en una rama local. Aquí solo lo usamos para probar la propuesta ajena. Volver a vuestra rama devuelve los archivos a vuestra propuesta.

**A:** desde **Files changed**, abrid **Review changes**, escribid al menos un comentario concreto sobre el cambio y el resultado de la prueba y enviad la revisión con **Submit review**. Si todo está correcto, elegid **Approve**. Si detectáis un problema real, elegid **Request changes** y comprobad la corrección antes de aprobar. Los nombres de los botones pueden variar ligeramente.

**B:** responded a la revisión. Después **A** integra la PR de B con **Create a merge commit / Merge pull request** y confirma la integración, no con squash ni rebase. Si esa opción está deshabilitada, A debe habilitar **Allow merge commits** en los ajustes del repositorio. No borréis aún las ramas, porque las usaréis para volver de las pruebas.

Guardad el enlace de la PR y la evidencia de revisión. No integréis todavía la PR de A.

## 3.3. Resolver el conflicto de la PR de A — A resuelve, B revisa

La PR de A debería indicar ahora que tiene conflictos con `main`. Si la interfaz tarda, recargad. En el nivel 1 resolvió B; ahora A practicará la resolución y B revisará el resultado.

**Antes de ejecutar los comandos, A responde y B contrasta:** ¿qué dos versiones intentaremos combinar y en qué rama quedará la resolución?

**A:** en terminal, sin usar el editor de conflictos de GitHub ni el botón de actualizar rama:

```bash
git switch propuesta/saludo-a
git fetch origin
git merge origin/main
```

Git se detendrá en `script.js`. Guardad una captura de `git status` y las marcas. En este caso, `HEAD` es la propuesta de A y la otra parte es el `main` que ya incluye la propuesta de B.

**Ambos:** acordad esta síntesis y dejad una única asignación:

```js
  mensaje.textContent = "¡Bienvenidas y bienvenidos! Construimos esta web revisando en equipo.";
```

**A:** eliminad todas las marcas y completad la resolución:

```bash
git diff
git diff --check
git add script.js
git diff --staged
git commit -m "Integrar main y acordar un saludo conjunto"
git push origin propuesta/saludo-a
```

Observad que **no habéis hecho push a `main`**. La PR existente se actualiza al subir commits a su rama; no hay que abrir otra.

**B:** actualizad la información del remoto y probad la propuesta resuelta:

```bash
git fetch origin
git switch --detach origin/propuesta/saludo-a
```

Abrid la web y pulsad el botón. Después volved a vuestra rama:

```bash
git switch propuesta/saludo-b
```

Revisad en GitHub el diff y los nuevos commits de la PR de A. Desde **Review changes**, dejad un comentario concreto sobre la resolución, enviad la aprobación y **B integra la PR de A con un commit de merge**.

> **Píldora: resolver en la rama de la propuesta.**
> Traer `origin/main` a la rama de A permite resolver y probar antes de integrar. El push actualiza esa rama remota y su PR, no `main`. La revisión debe comprobar el resultado final, también después de un conflicto.

## 3.4. Volver al resultado compartido — ambos

```bash
git switch main
git pull --no-rebase origin main
git fetch origin
git status
git log --oneline --graph --decorate --all
git rev-parse HEAD
```

Comprobad que las dos PR están integradas, que coincidís en el commit final y que la web conserva descripción, fondo, color, pie, adaptación móvil y saludo conjunto.

**Pregunta 4:** ¿qué diferencia hay entre hacer push de una rama, abrir una PR, aprobarla e integrarla? ¿Por qué fue necesario probar y revisar el resultado de la resolución antes de integrar la PR de A?

**Pregunta 5:** comparad el trabajo directo en `main`, las ramas con merge local y las PR. ¿Qué errores ayuda a detectar cada forma de trabajo? ¿Cuál elegiríais para un proyecto con más personas y por qué?

**🛑 Sincronización 8:** ambos tenéis el mismo resultado final y la web funciona.

## 3.5. Reto final: una mejora propia sin receta

Ahora aplicaréis el flujo sin una lista de comandos. Podéis consultar los apartados anteriores y el botiquín, pero debéis decidir qué ejecutar y en qué rama.

- Elegid **una mejora pequeña**: aclarar el texto del botón, mejorar el estilo del pie o añadir un estilo de foco visible para navegar con teclado. También podéis acordar otra de alcance similar. Conservad las mejoras anteriores y el saludo conjunto.
- **B desarrolla:** desde `main` actualizado, crea una rama nueva, implementa y prueba la mejora, hace commit, publica la rama y abre una PR hacia `main`.
- **B:** en la descripción explica qué cambia, por qué y **cómo comprobarlo**, con el resultado esperado.
- **A revisa:** lee el diff y prueba esa rama en su MV. Deja una revisión concreta; si pide cambios, B los corrige y A vuelve a comprobarlos. Cuando esté correcta, A aprueba e integra con un commit de merge.
- Ambos actualizáis vuestro `main` y comprobáis que tenéis el mismo commit, sin cambios pendientes, y que la web sigue funcionando.

Este reparto compensa la preparación inicial y la documentación, que asume A: al terminar la entrega, cada persona habrá creado dos PR y revisado e integrado otras dos.

No hace falta provocar otro conflicto ni entregar más capturas: **el enlace a esta PR, su descripción y su revisión** son la evidencia del reto.

**🛑 Sincronización 9:** mejora propia integrada y probada, revisión cruzada y ambos repositorios sincronizados.

---

# Entrega

## Cómo se evaluará

| Criterio | Peso | Qué debéis demostrar |
| --- | --- | --- |
| Configuración y trazabilidad | 10 % | Remoto compartido, identidad de ambos y acceso docente |
| Colaboración en `main` | 20 % | Distinguir rechazo y conflicto e integrar sin perder cambios |
| Ramas e integración local | 15 % | Separar mejoras y explicar la dirección de los merges |
| PR y revisión cruzada | 25 % | Proponer, probar y revisar; resolver en la rama correcta y aplicar el flujo al reto final |
| Comprensión y reflexión individual | 20 % | Respuestas razonadas y una reflexión propia por persona |
| Entrega y funcionamiento | 10 % | Evidencias útiles, documentación revisada, web funcional y sincronización |

**No se evalúa acumular capturas ni ejecutar comandos de memoria.** Se valora explicar las decisiones, comprobar el resultado y revisar de verdad el trabajo ajeno. Una aprobación sin pruebas ni comentarios concretos no demuestra una revisión. Se distinguirá entre trabajo autónomo y razonado, correcto con ayuda, parcial y no demostrado; la comprensión se contrastará también individualmente.

## Evidencias y respuestas

Una entrega por pareja, con aportaciones identificadas de ambas personas. Usad la [plantilla de entrega](docs/plantilla-entrega.md) facilitada con el material. Copiadla como `ENTREGA.md` en la raíz de vuestro repositorio; no hace falta copiar el resto de la guía.

Incluid:

1. **Enlace al repositorio compartido**, nombres o identificadores de la pareja y reparto A/B.
2. **Enlaces a las dos PR de saludo y a la PR del reto final**, donde se vean la revisión cruzada y su integración.
3. **Capturas legibles**, con una frase que explique cada una:
   - Rechazo del push de A en el apartado 1.1.
   - Conflicto de color antes de resolver: marcas y `git status`.
   - Gráfico tras integrar las ramas del nivel 2.
   - Conflicto del saludo en la rama de A y revisión de esa resolución en GitHub.
   - Web final y comprobación de que ambos tenéis el mismo commit y el directorio limpio.
4. **Respuestas a las preguntas 1–5**, con vuestras palabras y referencias a lo ocurrido. Repartid la redacción y señalad quién redacta cada respuesta; la otra persona la revisa.
5. **Reflexión individual de cada integrante:** un error o dificultad real, cómo lo investigaste y qué harías para colaborar mejor. Si no hubo errores adicionales, explica una duda y cómo la resolviste.

Las capturas pueden combinar varias evidencias si siguen siendo legibles. No hace falta capturar todos los comandos. Guardadlas en `evidencias/` y enlazadlas desde `ENTREGA.md`; evitad imágenes enormes o datos sensibles.

## Publicar la entrega con una última PR

- **A:** desde `main` actualizado, cread `docs/entrega`, incorporad `ENTREGA.md` y las capturas, revisad el contenido, haced commit y publicad la rama.
- **B:** aportad vuestras respuestas y reflexión a A para incluirlas. Revisad la PR de entrega, comprobad enlaces e imágenes y pedid correcciones si faltan evidencias. Cuando sea correcta, aprobad e integrad con un commit de merge.
- **Ambos:** una vez integrada la PR de documentación, cambiad a `main`, haced pull y comprobad que `git status` indica un directorio limpio y que `git rev-parse HEAD` coincide en ambos equipos. Esta comprobación es obligatoria, pero no tenéis que registrarla en `ENTREGA.md` ni abrir otra PR.
- Añadid el enlace de esta PR en la entrega del aula virtual, junto con el enlace al repositorio. Si es privado, comprobad el acceso del docente.

Esta última PR evita que la documentación se añada directamente a `main`. La captura de igualdad de commits puede corresponder al final del nivel 3, antes de incorporar la documentación; comprobad también la igualdad tras la entrega sin intentar añadir una captura nueva en un bucle infinito.

### Lista de comprobación

- [ ] Hay commits de ambas personas con identidad correcta.
- [ ] Se distingue el push rechazado del conflicto de integración.
- [ ] Se han integrado las dos mejoras del nivel 2.
- [ ] Las dos PR de saludo tienen revisión de la otra persona y están integradas.
- [ ] B ha resuelto el conflicto de color y A el del saludo, con revisión de la pareja.
- [ ] La resolución del saludo está en el historial de la rama de A.
- [ ] La mejora propia tiene una PR integrada con descripción, prueba y revisión concreta.
- [ ] La web funciona y no quedan marcas de conflicto.
- [ ] Están las capturas explicadas, las cinco respuestas y las dos reflexiones.
- [ ] La entrega también se ha incorporado mediante una PR revisada.
- [ ] Ambos repositorios locales están actualizados y limpios.

---

# Botiquín de Git

| Quiero… | Comando o acción |
| --- | --- |
| Saber dónde estoy y qué está pendiente | `git status` y `git branch --show-current` |
| Ver cambios sin preparar | `git diff` |
| Ver lo que entrará en el commit | `git diff --staged` |
| Ver el historial y las ramas conocidas | `git log --oneline --graph --decorate --all` |
| Actualizar mi información del remoto sin integrar | `git fetch origin` |
| Quitar un archivo de staging sin borrar su edición | `git restore --staged archivo` |
| Salir de un merge en conflicto y volver a antes del intento | `git merge --abort` |
| Salir del visor del historial | Pulsar `q` |

`merge --abort` es una vía de recuperación, no la resolución del ejercicio. Usadlo con ayuda si os habéis equivocado de rama y volved al paso correcto. Por eso empezamos cada integración con el directorio limpio.

Si Git abre un editor para un mensaje de merge, aceptad o editad el mensaje: en Nano, `Ctrl+O`, Intro y `Ctrl+X`; en Vim, `Esc`, `:wq` e Intro.

**Si algo no coincide con la guía:** parad, anotad el comando y el mensaje exactos, comprobad rama y estado y consultad con la pareja o el docente. No probéis comandos destructivos para «dejarlo como antes».

La [guía docente](docs/guia-docente.md) contiene preparación, resultados esperados, diagnóstico y criterios de evaluación; no es necesaria para seguir la práctica.

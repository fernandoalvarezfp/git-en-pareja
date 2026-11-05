# Guía docente · Git y GitHub en pareja

Esta guía acompaña al [enunciado del alumnado](../README.md). Incluye preparación, resultados esperados y orientaciones de evaluación. Puede distribuirse aparte si se prefiere que las soluciones no estén disponibles durante la práctica.

## 1. Enfoque y objetivos

El grupo ya ha practicado comandos en Learn Git Branching, pero tiene poca experiencia con archivos, terminal y GitHub reales. La progresión propuesta es:

1. **`main` compartida:** observar el rechazo de un push, integrar trabajo ajeno sin conflicto y resolver después un conflicto de contenido.
2. **Ramas:** aislar mejoras y comprender la dirección de un merge desde terminal.
3. **Pull requests:** aprender qué es una propuesta de integración en GitHub, revisar código de otra persona y resolver un conflicto en la rama de la propuesta.

No se presupone experiencia con PR. El enunciado introduce su propósito, base/compare, las pestañas principales y la diferencia entre publicar, abrir, aprobar, integrar y cerrar sin integrar. Conviene comprobar estas distinciones antes de crear la primera.

El reparto asegura práctica de recuperación e integración en ambas personas: A recibe el push rechazado e integra sin conflicto en el apartado 1.1; B recibe el rechazo y resuelve el color en el apartado 1.2; A resuelve el saludo en el nivel 3. La otra persona contrasta cada resultado. Las predicciones breves antes de fetch, pull y merge se comentan oralmente y se contrastan después, sin añadir evidencias a la entrega.

El reto final exige una mejora pequeña mediante rama y PR sin proporcionar los comandos: B desarrolla y A prueba, revisa e integra. La PR de documentación invierte esos papeles: A prepara y publica; B revisa e integra. Así cada persona crea dos PR y revisa e integra otras dos, compensando la preparación inicial que asume A.

Al terminar deberían poder distinguir directorio de trabajo, staging, commits locales, referencias remotas y ramas en GitHub; integrar sin sobrescribir a la pareja; justificar una resolución; y revisar un cambio antes de aceptarlo.

Los puntos de sincronización marcan el avance. Conviene comprobar cada hito antes de permitir el siguiente, especialmente cuando ambas personas deben partir del mismo commit.

## 2. Preparación del material y del entorno

### Material que se facilita

```text
README.md                   Enunciado del alumnado
index.html                  Web inicial
style.css
script.js
.gitignore                  Exclusiones opcionales para el material
docs/
  plantilla-entrega.md      Modelo que copiarán como ENTREGA.md
  guia-docente.md           Este documento
```

Los archivos web no tienen dependencias externas. Se abren directamente en el navegador. El CSS incluye una única variable de color y el JS una única asignación de saludo: son los puntos de conflicto controlado.

Podéis publicar el material en un repositorio docente o distribuir un ZIP desde el aula virtual. Si distribuís una carpeta, **no incluyáis el directorio `.git`**. El alumnado debe crear un repositorio nuevo y vacío, no continuar el historial docente.

El enunciado pide que A copie **solo los tres archivos web** al repositorio de la pareja. Así el primer commit es pequeño y resulta más fácil observar los cambios. La plantilla y las capturas se incorporan al final. La guía se consulta fuera del repositorio de trabajo.

Si el material se publica en GitHub, indicad que descarguen el ZIP: en esta práctica **no deben usar Fork ni Use this template**, porque queremos practicar explícitamente `init`, creación del remoto y `clone`.

### Requisitos a comprobar

- Dos MV Debian por pareja, con terminal, editor, navegador, Git y conectividad a GitHub.
- Una cuenta de GitHub por persona y un método de autenticación Git funcional.
- Posibilidad de crear un repositorio y añadir a la otra persona como colaboradora con escritura.
- Convenciones del centro para visibilidad, correos de autoría, cuentas y tratamiento de capturas.
- Acceso docente a repositorios privados.
- Rama principal llamada `main`; el enunciado usa `git init -b main`.
- Las cuentas y políticas de GitHub permiten integrar con **merge commit**.

No activéis protección de `main` que impida los pushes de los niveles 1 y 2. Tampoco configuréis historial lineal obligatorio, porque la práctica conserva merges. Tras el ejercicio se puede debatir qué protecciones tendría un proyecto real.

La práctica no exige checks automáticos ni desplegar la web. Una marca verde de «puede integrarse» no certifica que el código funcione.

### Autenticación: punto especialmente sensible

Anticipad la diferencia entre la identidad del commit y las credenciales. Los correos `noreply` permiten evitar exponer correos personales.

Para HTTPS, las contraseñas de cuenta no sirven. Si hay un mecanismo institucional de credenciales, priorizadlo. Los tokens *fine-grained* tienen restricciones de propiedad y acceso que pueden complicar el caso de B como colaborador de un repositorio propiedad de A; no asumáis que seleccionar permisos de contenido basta. SSH previamente configurado o un token clásico autorizado por el centro son alternativas.

No pidáis al alumnado que os enseñe tokens ni que los comparta con la pareja. No uséis tokens dentro de la URL ni `credential.helper store`. Si falla la autenticación, separad esa incidencia del contenido de Git: no se resuelve rehaciendo el historial.

## 3. Mapa de hitos y resultados esperados

| Punto | Condición de avance | Evidencia útil |
| --- | --- | --- |
| 0 | Mismo commit inicial en `main`, web funcional y acceso al remoto | `status`, `remote -v`, `log` |
| 1 | A y B tienen un commit distinto aún sin publicar | `log -1` en cada equipo |
| 2 | A ha integrado ambos cambios sin conflicto y B se ha actualizado | Push rechazado de A y gráfico de merge |
| 3 | A y B tienen colores distintos confirmados desde una base común | Diff de cada commit |
| 4 | Merge resuelto con `#b45309`, mismo commit en ambos | Marcas anteriores a resolver, `status` limpio y `rev-parse HEAD` |
| 5 | Dos ramas de mejora publicadas; `main` aún sin las mejoras | Selector de ramas de GitHub |
| 6 | Ambas mejoras integradas localmente y publicadas | Gráfico con los merges de móvil y pie |
| 7 | Dos PR de saludo abiertas desde una base común | Base `main` y dos ramas compare distintas |
| 8 | PR de B y después PR de A integradas y revisadas; locales actualizados | Conversaciones de PR y web final |
| 9 | Mejora propia integrada mediante PR y locales sincronizados | Descripción con prueba y revisión concreta de la PR del reto |

Las capturas de `HEAD` iguales se interpretan junto con el `status` limpio. Tener el mismo commit no impide que haya archivos modificados o sin seguimiento en una de las máquinas.

## 4. Por qué los conflictos son reproducibles

### Nivel 1: rechazo sin conflicto

A cambia el HTML y B cambia el CSS, ambos desde la base inicial y antes de publicar. B publica primero; el push de A se rechaza porque el remoto ya contiene un commit que A aún no tiene.

Después de `fetch`, el gráfico debería tener esta forma conceptual:

```text
      B1  ← origin/main
     /
I ──┤
     \
      A1  ← main de A
```

A hace merge de `origin/main` a su `main`. Como son archivos distintos, el merge es automático, conserva ambos commits y genera un commit con dos padres. A lo publica y B avanza hasta él.

**No asociar rechazo con conflicto:** el servidor rechazó una actualización de historial; el contenido se combinó después sin intervención manual.

### Nivel 1: conflicto de color

Tras sincronizarse, ambas personas cambian la misma línea de la variable CSS y hacen commit antes de publicar. Esta vez A publica primero. El pull de B descarga y trata de integrar historias divergentes; Git no puede escoger entre dos valores para la misma línea.

B verá su verde en `HEAD` y el violeta de A en el otro bloque. El resultado acordado es ámbar:

```css
  --color-principal: #b45309;
```

La resolución no exige conservar literalmente una de las propuestas: puede ser una tercera opción, si se acuerda y funciona. Deben permanecer una sola variable y ninguna marca de conflicto.

### Nivel 2: ramas sin conflicto

A añade un pie en HTML; B añade una media query al final del CSS. B integra móvil primero. A actualiza `main` antes de integrar el pie. Los merges con `--no-ff` dejan visibles los puntos de integración.

Las ramas no eliminan los conflictos por sí mismas. Aquí no los hay porque los cambios son compatibles y la publicación se ordena explícitamente.

### Nivel 3: conflicto en una PR

Las dos ramas parten del mismo `main` y cambian la misma asignación del saludo. Ambas PR pueden aparecer inicialmente como integrables. A revisa e integra primero la de B: esto cambia la base y hace que la de A entre en conflicto.

A incorpora `origin/main` en `propuesta/saludo-a`, no al revés. `HEAD` contiene el saludo de A y el otro bloque contiene el saludo de B, ya integrado en `main`. El merge se resuelve en la rama de A y el push actualiza la PR existente. B prueba y revisa ese resultado antes de integrar. Así A también resuelve un conflicto personalmente.

Conservad las ramas hasta terminar las pruebas cruzadas: el enunciado vuelve a ellas después de las inspecciones con `--detach`.

Resultado acordado:

```js
  mensaje.textContent = "¡Bienvenidas y bienvenidos! Construimos esta web revisando en equipo.";
```

Los merges realizados en GitHub no actualizan mágicamente las ramas locales: ambos deben volver a `main` y hacer pull.

## 5. Orientaciones para las preguntas

Son ideas esperadas, no textos que deban memorizar.

### Predicciones orales

No se califican por acertar antes de probar. Pedid una justificación y contrastadla con el resultado:

- Antes del fetch del nivel 1, A anticipa que se actualizará `origin/main`, no su `main` ni los archivos de trabajo. El merge posterior será automático porque los cambios afectan a archivos distintos.
- Antes del pull del conflicto de color, ambos distinguen el rechazo de historial del conflicto de contenido. Anticipan un conflicto al integrar las modificaciones incompatibles de la misma línea.
- Antes del merge de móvil, B identifica `main` como rama receptora y explica que `--no-ff` obliga a conservar un commit de merge aunque sea posible un avance directo.
- Antes de resolver el saludo, A identifica su rama como destino de la resolución y el `main` que contiene la propuesta de B como historial que incorporará.

La pareja contrasta cada predicción para evitar que una sola persona razone por ambas.

### Pregunta 1

- El push de A no puede avanzar el remoto sin incorporar el commit publicado por B.
- `fetch` actualiza las referencias remotas conocidas, sin modificar su rama de trabajo.
- `log --oneline --graph --decorate --all` permite observar la divergencia: `main` y `origin/main` contienen commits distintos desde una base común.
- `merge` integra el historial descargado en la rama actual; aquí combina automáticamente HTML y CSS.
- `push` publica el resultado integrado.
- Cambiar archivos distintos reduce conflictos de contenido, pero no elimina la necesidad de sincronizar historias.

### Pregunta 2

- En ambos casos divergen las historias y el push es rechazado.
- Solo el segundo tiene cambios incompatibles en el mismo lugar.
- El conflicto aparece al integrar, no durante el push.
- Editar resuelve el contenido; `add` marca el archivo como resuelto; `commit` completa el merge.
- Las marcas son indicadores temporales de Git, no código válido que deba publicarse.

### Pregunta 3

- Cada rama tiene su propia referencia; publicarla no integra sus commits en otra.
- El merge afecta a la rama actual, que debe ser `main` en estas integraciones.
- Actualizar antes permite partir del trabajo ya publicado por B y publicar una historia que lo contiene.
- `--no-ff` hace visible la integración incluso si podría avanzar la referencia sin merge adicional.

### Pregunta 4

- Push publica commits y actualiza la rama remota.
- Abrir una PR crea una propuesta de integración en GitHub.
- Aprobar expresa una revisión favorable; no modifica por sí solo la base.
- Integrar incorpora los cambios a la base elegida.
- Resolver un conflicto cambia código: hay que volver a probarlo y revisarlo. Un merge sin marcas puede seguir conteniendo un error lógico.

### Pregunta 5

- `main` compartida requiere mucha coordinación y expone inmediatamente los cambios integrados.
- Ramas permiten separar propuestas, pero siguen necesitando pruebas e integración cuidadosa.
- PR añaden conversación, revisión trazable y una oportunidad de detectar problemas antes de incorporarlos a `main`.
- No existe un flujo que garantice ausencia de errores. Se valora una elección razonada según tamaño del equipo y necesidad de revisión, no solo responder «PR».

## 6. Diagnóstico y recuperación

Antes de recomendar nada, pedid el comando exacto, el mensaje completo, `git status`, `git branch --show-current` y, si procede, el gráfico. No hagáis `reset --hard` ni force push para rescatar una pareja: eliminan precisamente el trabajo que están aprendiendo a conservar.

| Síntoma | Causa probable | Intervención |
| --- | --- | --- |
| `Permission denied`, 403 o autenticación fallida | Credenciales, invitación pendiente, permisos del token o URL de otro repositorio | Verificar cuenta, URL y acceso; no tocar los commits |
| El remoto ya contiene un commit inicial ajeno | Se marcó crear README/licencia en GitHub | Antes de empezar, usar un remoto realmente vacío. Si ya hay trabajo, revisarlo con el docente; no sugerir `--allow-unrelated-histories` a ciegas |
| A no consigue provocar el primer rechazo | B aún no publicó, A ya incorporó el commit de B o el push apunta a otro remoto o rama | Comprobar la publicación de B, el historial, el remoto y la rama de destino; si hace falta, crear una nueva ronda equivalente con cambios independientes. No haber confirmado el cambio local no impide por sí solo que el push sea rechazado |
| No aparece el conflicto de color | Una persona partió del nuevo color de la otra o editó otra línea | Sincronizar, escoger una nueva base y repetir ambos con valores distintos desde esa base, confirmando antes de publicar |
| El push funciona pero la web de la pareja no cambia | Subió a otra rama, a otro remoto o la pareja no hizo pull | Verificar rama actual, destino y actualización del otro local |
| `Your local changes would be overwritten` | Trabajo sin confirmar antes de cambiar o integrar | Revisar y confirmar el trabajo correcto en su rama antes de continuar; no borrarlo por defecto |
| `MERGE_HEAD exists` o Git indica merge en curso | No se completó la resolución anterior | Revisar `status`, resolver, `add` y `commit`; no iniciar otra integración |
| `git diff` no muestra nada, pero el cambio existe | El archivo ya está preparado | Usar `git diff --staged` |
| La PR de A no tiene conflicto después de integrar B | A creó su rama después, cambió otra línea o ya incorporó B | Comprobar bases y commits. Si hace falta, repetir una pareja de propuestas desde un `main` común sin reescribir el trabajo anterior |
| No se puede aprobar o integrar | Autor intentando aprobar su propia PR, permisos o reglas del repositorio | Intercambiar al revisor correcto y comprobar acceso de colaboración y ajustes |
| El saludo no cambia al probar la otra rama | Archivo sin guardar, pestaña apuntando a otra carpeta o caché | Confirmar ruta, recargar y comprobar consola del navegador |
| La PR sigue mostrando conflicto tras resolver | Resolución sin publicar o push a otra rama | Verificar el commit y el push a `propuesta/saludo-a` |
| `git branch -d` impide borrar una rama tras una PR | Metadatos de seguimiento o integración no actualizados | Hacer fetch, comprobar el historial y omitir la limpieza si hay dudas; no usar `-D` por rutina |

### Abortar un merge

Con el directorio limpio antes de la operación, `git merge --abort` permite volver al punto previo si se intentó integrar desde la rama equivocada. Primero confirmad el estado. Abortar no completa el objetivo: después hay que repetir correctamente.

### Si una persona se adelanta

No es necesario reconstruir toda la práctica ni borrar commits. Identificad qué objetivo dejó de observarse y preparad una ronda adicional equivalente desde una base común. Por ejemplo, para repetir el conflicto de color, ambos parten del color vigente y proponen dos colores nuevos en la misma línea. Documentad brevemente la incidencia en la reflexión.

### Archivos y datos sensibles

Si una captura muestra un token real, revocadlo antes de cualquier otra cosa. Borrar el archivo en un commit posterior no lo elimina del historial. Revisad el incidente con el docente y aplicad el procedimiento del centro para retirar el secreto del historial cuando corresponda.

## 7. Evaluación propuesta

| Criterio | Peso | Qué observar |
| --- | --- | --- |
| Configuración y trazabilidad | 10 % | Repositorio compartido correcto, commits identificables de ambos y acceso docente |
| Colaboración en `main` | 20 % | Rechazo sin conflicto, integración y resolución del color sin perder cambios |
| Ramas e integración local | 15 % | Mejoras aisladas y merges con dirección y orden correctos |
| PR y revisión cruzada | 25 % | Base/compare correctos, pruebas, comentarios concretos, revisión de la resolución y transferencia al reto final |
| Comprensión y reflexión individual | 20 % | Cinco respuestas razonadas y una reflexión propia por persona |
| Entrega y funcionamiento | 10 % | Enlaces y capturas útiles, PR de documentación, web final y sincronización |

El resumen de estos criterios y pesos también aparece en el enunciado para que el alumnado conozca qué se evalúa antes de entregar.

Una captura del botón de aprobar no prueba una buena revisión. Buscad referencias al diff, al comportamiento observado o a una decisión de integración. No es necesario inventar objeciones: una aprobación puede ser rigurosa si explica qué se comprobó.

En el reto final, valorad que B elija la rama y el destino correctos y describa una prueba reproducible, y que A pruebe y revise el resultado. No se exige memorizar comandos ni una mejora compleja: pueden consultar la guía y pedir ayuda. El enlace y la actividad de la PR bastan como evidencia; no se piden capturas adicionales.

La comprobación de sincronización posterior a integrar la documentación es obligatoria, pero se realiza fuera de `ENTREGA.md`: no se exige una casilla ni otra captura o PR para registrarla. Así la entrega no afirma por adelantado una comprobación futura ni obliga a encadenar actualizaciones.

Dentro de cada criterio, podéis distinguir entre **autónomo y razonado**, **correcto con ayuda**, **parcial** y **no demostrado**. El producto se evalúa en pareja, pero la comprensión debe contrastarse individualmente.

Para comprobar autoría e historial:

```bash
git log --all --format='%h %an <%ae> %s'
git log --oneline --graph --decorate --all
```

Contrastad esos datos con las PR, las respuestas y una breve explicación individual. Un único commit grande al final no demuestra el recorrido.

## 8. Cierre y ampliaciones opcionales

Solo tras completar la práctica:

- Activar protección de `main` y discutir por qué el nivel 1 ya no sería posible con esas reglas.
- Borrar ramas ya integradas desde GitHub y observar `git fetch --prune` localmente. Explicar que borrar una rama no borra los commits que ya forman parte de `main`.
- Comparar el historial obtenido con squash o rebase en otro repositorio de pruebas; no reescribir el de esta entrega.
- Añadir una comprobación automática sencilla en una PR y distinguirla de la revisión humana.
- Ampliar el reto final con una issue que describa la necesidad y unos criterios de aceptación antes de abrir la rama y la PR.

# BOOTCAMP FULLSTACK 2022 - Git

## Comandos

### Helpers

```sh
# Con esto comprobáis que os funciona git
git --version

# Ver comandos básicos de git
git help

# Ver más comandos de git (cuidado que son muchos)
git help -a
```

### Fundamentales (local)

> RECORADAR LOS 3 ESPACIOS DE TRABAJO EN GIT
>
>   STAGED -> lo que se va a committear (aparece en verdecito en el git bash)
>
>   UNSTAGED or NOT STAGED -> cambios que tenemos en ficheros ya trackeados
>
>   UNTRACKED -> ficheros que todavía no han sido trackeados

```sh
# Estado actual del espacio de trabajo git
git status

# Ver el historial de commits
git log
git log --oneline

# Añadir los cambios de un fichero para commitear
git add <nombre-ficheros>

# Hacer un commit de que lo añadimos en nuestro STAGED con el git add para commitear
git commit -m "<mensaje de commit>"

# Crear nueva rama y cambiarse a ella
git checkout -b <nombre-rama>

# Cambiarse de rama
git checkout <nombre-rama>

# Para mergear una rama
# Para poder mergearla la rama que queremos mergear tiene que tener commits
git checkout <rama-en-la-que-queremos-que-se-mergee-otra-rama>
git merge <rama-que-queremos-mergear>

# Consultar las ramas de local
git branch

# Consultar las ramas de local y remoto
git branch -a

# Deshacer uno o varios commits sin perder los cambios con 'n' el número de commits a deshacer
git reset HEAD~n

# Quitar de STAGED ficheros
git restore --staged <nombre-ficheros>

# Guardar en stash cambios para poder sacarlos luego
git stash

# Sacar los últimos cambios guardados en el stash
git stash pop
```

### Comandos destructivos (local) !!!!!!CUIDADO!!!!!!
```sh
# Eliminar cambios en un fichero
git restore <nombre-ficheros>

# Eliminar ficheros untracked
git clean -fd

# Deshacer uno o varios commits sin PERDIENDO LOS CAMBIOS con 'n' el número de commits a deshacer
git reset --hard HEAD~n

# Eliminar una rama de local
git branch -d <nombre-rama>
```

### Fundamentales (remoto)
```sh
# Subir tus cambios al remoto
git push

# Traer cambios de remoto a local
git pull

# Buscar cambios en remoto
git fetch

# Cuando hacéis un merge puede que un futuro os aparezca un mensaje de que hubo conflictos
# Los IDEs ya proporcionan herramientas para solucionarlos
```

### Comandos destructivos (remoto) !!!!!!CUIDADO!!!!!!
```sh
# Eliminar una rama de remoto
git push --delete origin <branch-name>
```
# BOOTCAMP FULLSTACK 2022

## ¿Cómo generar un el proyecto base?

Esto debería hacerlo SOLO UNA PERSONA del equipo con todo el equipo presente y una vez terminado el resto podrán clonar el repositorio
y así todos tendréis el mismo proyecto inicial.

### 1. Crear repository de Github

Aquí es donde subiremos el código fuente del proyecto.

Todos debéis tener una cuenta de Github con vuestro correo de campusdual (si ya teníais podéis usar esa pero que se reconozca que sois vosotros). El que cree el repositorio
debe dar acceso al resto de los miembros del equipo.

![First step](https://i.imgur.com/POsfrpu.png)
![Second step](https://i.imgur.com/ZwojDsk.png)
![Third step](https://i.imgur.com/4r2rMNL.png)
![Fourth step](https://i.imgur.com/ug4UMFZ.png)
![Fifth step](https://i.imgur.com/5eXsJws.png)

### 2. Ontimize boot backend

Ejecutar el comando, cambiando <YOUR_GROUP_ID> <YOUR_ARTIFACT_ID> <YOUR_VERSION> <GROUP_ID.ARTIFACTID> por vuestro nombre de grupo y por el nombre de vuestro proyecto. **Recordad que el artifactId se llame exactamente igual que vuestro respositorio clonado de github, solo así se creará todo en el sitio que corresponde.**

```
mvn archetype:generate -DgroupId=YOUR_GROUP_ID -DartifactId=YOUR_ARTIFACT_ID -Dversion=YOUR_VERSION -Dpackage=YOUR.GROUPID.ARTIFACTID -DarchetypeGroupId=com.ontimize -DarchetypeArtifactId=ontimize-boot-backend-archetype -DarchetypeVersion=1.0.2 -DinteractiveMode=false
```
![Sixth step](https://i.imgur.com/HxkisoO.jpeg)

Tras esto tendréis en vuestra carpeta de proyecto algo similar a esto:
![Seventh step](https://i.imgur.com/idHe64H.png)

> source: https://ontimize.github.io/ontimize-boot/getting_started/

Ahora solo queda subir al repositorio esto:
![Eighth step](https://i.imgur.com/nlzjLmr.jpeg)

### 3. Ontimize web

Clonaremos el proyecto base de Ontimize Web dentro de nuestro proyecto del back de Ontimize. 
![Ninth step](https://i.imgur.com/eebmS36.png)
![Tenth step](https://i.imgur.com/uN6AUWC.png)
![Eleventh step](https://i.imgur.com/Ip4ISXM.png)
![Twelveth step](https://i.imgur.com/JHBh1St.png)
![Twelveth step](https://i.imgur.com/7BnOiuw.jpeg)

### 4. Datos para la base de datos de remoto

- BD: PostgreSQL
- IP: 45.84.210.174
- Port: 65432

## Enlaces interesantes

### Sobre lambdas en Java

- [Get a Taste of Lambdas and Get Addicted to Streams by Venkat Subramaniam](https://www.youtube.com/watch?v=1OpAgZvYXLQ)
- [Exploring Collectors by Venkat Subramaniam](https://www.youtube.com/watch?v=pGroX3gmeP8)
- [Functional Programming with Java 8 by Venkat Subramaniam](https://www.youtube.com/watch?v=15X0qFtBqiQ)


### Metodologías ágiles

- [Planning poker free](https://www.planitpoker.com/)
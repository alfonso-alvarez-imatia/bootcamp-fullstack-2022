# BOOTCAMP FULLSTACK 2022

## FAQ

### Error lanzando la aplicación de Ontimize Web (npm start)

Si tenéis un error como este:

```js
chunk {vendor} vendor.js, vendor.js.map (vendor) 347 kB [initial] [rendered]
Date: 2022-06-27T12:40:29.494Z - Hash: 35f9ace13f4c0a65716c - Time: 12865ms
ERROR in node_modules/ontimize-web-ngx/lib/components/o-service-base-component.class.d.ts:101:40 - error TS2314: Generic type 'AbstractOServiceBaseComponent<T>' requires 1 type argument(s).
101     static ngBaseDef: ɵngcc0.ɵɵBaseDef<AbstractOServiceBaseComponent>;
                                           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
node_modules/ontimize-web-ngx/lib/components/o-service-component.class.d.ts:119:40 - error TS2314: Generic type 'AbstractOServiceComponent<T>' requires 1 type argument(s).
119     static ngBaseDef: ɵngcc0.ɵɵBaseDef<AbstractOServiceComponent>;
                                           ~~~~~~~~~~~~~~~~~~~~~~~~~
** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
```

Debéis lanzar el siguiente comando `npm cache clean --force` y eliminar la carpeta de `node_modules`.
Hecho esto volvemos a lanzar el `npm install` y luego el `npm start` y ya debería funcionar.

### ¿Cómo hacer debug `mvn spring-boot:run`?

Para IntelliJ hay que meter esto como parámetros en la *Run Configuration* de Maven:
`"-Dspring-boot.run.jvmArguments=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"`. Luego solo hay que lanzar en modo debug y en log de
la consola nos aparecerá una nueva seccción con una opción de *Attach Debugger*, lo presionamos y ya podemos hacer *debugging*.

TODO: PARA ECLIPSE.

### ¿Cómo hacer debug en el VSCode para Angular?

Se debe descargar la extensión *Debugger for Chrome*, como está deprecada hay que instalar la extensión JavaScript Debugger. Luego hay que ir a la sección de
*Run and Debug* que está en el menú lateral del VSCode. Hay habrá una opción para crear un `launch.json`. Esto nos creará una carpeta llamada .vscode y dentro
aparecerá el fichero. Luego hay que meter está configuración:

```json
{
    "type": "chrome",
    "request": "launch",
    "name": "Launch Chrome against localhost",
    "url": "http://localhost:4200",
    "webRoot": "${workspaceFolder}"
}
```

Dentro del *array* de *configurations*. Hecho esto solo hay que lanzar la aplicación normal con `npm start` y luego **ejecutar la nueva configuración**
desde la sección de *Run and Debug*.

### No puedo hacer operaciones *Create*, *Update* y *Delete* sobre usuarios, ¿qué ocurre?

Esto se debe a que el UserDao.xml de usuario tiene en las etiquetas `<GeneratedKey>USER_</GeneratedKey>`, esto solo sirve si el campo **USER_** es autogenerado, que no es el caso de *User*. Para que funcione hay que eliminar esas etiquetas.

### No me funciona la parte de Insertar una nueva entidad en el frontal, la vista me funciona mal

Esto se debe a que en el router nunca debe de aparecer el **:ID** antes del **new**, como véis en el siguiente ejemplo:

```json
const routes: Routes = [
{
  path: '',
  component: XHomeComponent
},
{
  path: "new",
  component: XNewComponent
},
{
  path: ":ID",
  component: XDetailComponent
}];
```

### ¿Cómo hacer los permisos?

Para gestionar permisos se puede ver la siguiente documentación:

- [https://ontimizeweb.github.io/docs/v8/guide/appconfig/#application-configuration](¿Cómo hacer que empiecen a funcionar los permisos de Ontimize?)
- [https://ontimizeweb.github.io/docs/v8/guide/permissions/](Explicación permisos Ontimize)

Hay varios pasos para que funcionen los permisos (si tenéis dudas el grupo de RitaPOP ya es experto ;) ). El primero de todos es
hacer que el frontal consulte los permisos de nuestro backend, para eso tenemos que cambiar nuestro app.config.ts y meter la configuración necesaria.

```json
  [OTRAS COSAS AQUI],
  permissionsServiceType: 'OntimizeEEPermissions',
  permissionsConfiguration: {
    service: 'permission'
  }
```

A partir de este momento se van a realizar consultas al *backend* para cargar permisos al endpoint `/loadPermissions`, entonces necesitamos crear ese
API REST en nuestro *back* y devolver los permisos que tendrá un rol.

Antes de hacerlo necesitamos los permisos en una tabla, para ello tenemos que modificar la tabla de *TROLE* y meter una nueva columna,
`JSON_ROLE_APP_PERMISSIONS VARCHAR(16777216)` (si os da error crear una nueva tabla y relacionádla con *TROLE*). En esa columna váis a meter un JSON de configuración
de permisos según se indica en [https://ontimizeweb.github.io/docs/v8/guide/permissions/](este enlace). Para poder meterlo necesitáis escapar las comillas dobles y quitar los saltos de línea `\"\"` para que se puedan meter en base de datos.

Una vez creada esa nueva columna necesitamos craer un nuevo endpoint en un nuevo controller, tal que así:

```java
@RestController
@RequestMapping("")
public class PermissionController extends ORestController<IPermissionsService> {

  @Autowired
  IPermissionsService permissions;

  @Override
  public IPermissionsService getService() {
    return this.permissions;
  }

  @RequestMapping(value = "/loadPermissions", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
  public ResponseEntity<EntityResult> loadPermissions() {
    return new ResponseEntity<>(permissions.getPermissionsByUser(), HttpStatus.OK);
  }
}
```

Creado esto vemos que nos hace falta un nuevo servicio de permisos `IPermissionsService` con un método `getPermissionsByUser()`. Lo definimos en API:

```java
public interface IPermissionsService {
  EntityResult getPermissionsByUser();
}
```

Ahora tenemos que crear la implementación en el MODEL. Aquí está un pequeño ejemplo de como se haría, pero tenéis que cambiarlo según os haga falta.

```java
  [OTRAS COSAS]


  @Autowired
  DefaultOntimizeDaoHelper daoHelper;

  @Autowired
  RoleDao roleDao;

  @Override
  public EntityResult getPermissionsByUser() {
    // el usuario autenticado
    // hacer la consulta a la tabla de permisos
    // sacar el array de permisos, de ahí se saca el primero de todos
    // se busca en base de datos a través del rol los permisos
    // Se mete el resultado en un entityResult y se devuelve
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    String rol = authentication.getAuthorities().toArray()[0].toString();
    EntityResultMapImpl entityResultMap = new EntityResultMapImpl();
    List<String> attr = new ArrayList<>();
    attr.add("JSON_ROLE_APP_PERMISSIONS");
    Map<String, Object> map = new HashMap<>();
    map.put("ROLENAME", rol);
    EntityResult query = this.daoHelper.query(this.roleDao, map, attr);
    entityResultMap.addRecord(new HashMap<>(){{put("permission",((List) query.get("JSON_ROLE_APP_PERMISSIONS")).get(0));}});
    return entityResultMap;
  }
```

Con esto ya debería funcionaros. Ahora cada vez que queráis cambiar los permisos tenéis que modificar el objeto de permisos que hayáis insertado en la base de datos
tal y como está definido en [https://ontimizeweb.github.io/docs/v8/guide/permissions/](este enlace).

### Bug con VSCode y npm start

Cuando se lanza la aplicación de Angular puede que se queden archivos corruptos al hacer un `npm start` o `npm install` mientras el VSCode está compilando el código, aparece
abajo a la izquierda del VSCode un icono indicando que se está compilando Angular. Esperar a que acabe el VSCode para después ejecutar desde terminal. 

## Equipos de trabajo

## Formaciones interesantes

- GIT en equipo (Jueves 29)
- HTTP (Tipos de peticiones, Headers, ...)
- Tema estilos (CSS, SCSS)
- RxJS
- Testing (back y front)
- Docker
- Angular (ver que más de angular puedo dar)
- JavaScript y TypeScript [event loop]

### Grupo 2 - Magic & CO

| Nombre | Correo |
|--------|--------|
| JOSE IGNACIO DIGON TELLERIA | joseignacio.digon@gmail.com |
| PABLO REY TRILLO | a10pablort@gmail.com |
| PAVEL KOVALEV | grandline.supervisor@gmail.com |
| ALEJANDRO GARCÍA SERANTES | alexg3110@gmail.com |
| MIRZA TURKOVIC IMAMOVIC | mirza.turkovicIm@gmail.com |

Orden de las *dailies*:

1. JOSE IGNACIO DIGON TELLERIA
2. PAVEL KOVALEV
3. PABLO REY TRILLO
4. ALEJANDRO GARCÍA SERANTES
5. MIRZA TURKOVIC IMAMOVIC

### Grupo 3 - Jaci

| Nombre | Correo |
|--------|--------|
| JAVIER FERNÁNDEZ MÉNDEZ	| fernandezmendez99@gmail.com |
| AMALIA AURORA VILLEGAS ASTUDILLO | astudillov.a@gmail.com |
| IGNACIO RODRIGUEZ EYRE | ignaciorguezeyre@gmail.com |
| CARLOS GARCIA BERMEJO | carlosgb3000@gmail.com |

Orden de las *dailies*:

1. JAVIER FERNÁNDEZ MÉNDEZ
2. CARLOS GARCIA BERMEJO
3. IGNACIO RODRIGUEZ EYRE
4. AMALIA AURORA VILLEGAS ASTUDILLO

### Grupo 4 - RitaPOP

| Nombre | Correo |
|--------|--------|
| JAVIER FANDIÑO CASAL | javierfandinho@gmail.com |
| ADRIAN CARNEIRO DIZ | adriancarndi@gmail.com |
| CINTA TAFUR CERREJON | cintatafur@gmail.com |
| TOMAS CARRALERO CORREDERAS | tcaralero@gmail.com |
| DONATO RAMOS MARTINEZ | donatoramos@hotmail.com |

Orden de las *dailies*:

1. ADRIAN CARNEIRO DIZ
2. CINTA TAFUR CERREJON
3. JAVIER FANDIÑO CASAL
4. TOMAS CARRALERO CORREDERAS
5. DONATO RAMOS MARTINEZ

## Shortcuts Github

- `Ctrl + k`: consola de acciones.
- `.`: entorno desarrollo de un repositorio.

## Shortcuts Jira

- `.`: consola de acciones.

## Enlaces interesantes

### Sobre lambdas en Java

- [Get a Taste of Lambdas and Get Addicted to Streams by Venkat Subramaniam](https://www.youtube.com/watch?v=1OpAgZvYXLQ)
- [Exploring Collectors by Venkat Subramaniam](https://www.youtube.com/watch?v=pGroX3gmeP8)
- [Functional Programming with Java 8 by Venkat Subramaniam](https://www.youtube.com/watch?v=15X0qFtBqiQ)

### Metodologías ágiles

- [Planning poker free](https://www.planitpoker.com/)

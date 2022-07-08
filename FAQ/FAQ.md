# BOOTCAMP FULLSTACK 2022 - FAQ

## Error lanzando la aplicación de Ontimize Web (npm start)

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

## ¿Cómo hacer debug `mvn spring-boot:run`?

Para IntelliJ hay que meter esto como parámetros en la *Run Configuration* de Maven:
`"-Dspring-boot.run.jvmArguments=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"`. Luego solo hay que lanzar en modo debug y en log de
la consola nos aparecerá una nueva seccción con una opción de *Attach Debugger*, lo presionamos y ya podemos hacer *debugging*.

TODO: PARA ECLIPSE.

## ¿Cómo hacer debug en el VSCode para Angular?

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

## No puedo hacer operaciones *Create*, *Update* y *Delete* sobre usuarios, ¿qué ocurre?

Esto se debe a que el UserDao.xml de usuario tiene en las etiquetas `<GeneratedKey>USER_</GeneratedKey>`, esto solo sirve si el campo **USER_** es autogenerado, que no es el caso de *User*. Para que funcione hay que eliminar esas etiquetas.

## No me funciona la parte de Insertar una nueva entidad en el frontal, la vista me funciona mal

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

## ¿Cómo hacer los permisos?

Para gestionar permisos se puede ver la siguiente documentación:

- [¿Cómo hacer que empiecen a funcionar los permisos de Ontimize?](https://ontimizeweb.github.io/docs/v8/guide/appconfig/#application-configuration)
- [Explicación permisos Ontimize](https://ontimizeweb.github.io/docs/v8/guide/permissions/)

Hay varios pasos para que funcionen los permisos (si tenéis dudas el grupo de RitaPOP ya es experto ;) ). Lo primero de todos es
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
de permisos según se indica en [este enlace](https://ontimizeweb.github.io/docs/v8/guide/permissions/). Para poder meterlo necesitáis escapar las comillas dobles y quitar los saltos de línea `\"\"` para que se puedan meter en base de datos.

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
tal y como está definido en [este enlace](https://ontimizeweb.github.io/docs/v8/guide/permissions/).

## Bug con VSCode y npm start

Cuando se lanza la aplicación de Angular puede que se queden archivos corruptos al hacer un `npm start` o `npm install` mientras el VSCode está compilando el código, aparece
abajo a la izquierda del VSCode un icono indicando que se está compilando Angular. Esperar a que acabe el VSCode para después ejecutar desde terminal. 

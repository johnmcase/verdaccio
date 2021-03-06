---
id: packages
title: "Acceso a Paquetes"
---
Es una serie de restricciones que permiten o restringen el acceso al almacenamiento local basado en unos criterios específicos.

Las restricciones de seguridad permanecen en los hombros de la extensión usada, por defecto `verdaccio`usa [htpasswd plugin](https://github.com/verdaccio/verdaccio-htpasswd). Si usas una extensión diferente ten en cuenta que el comportamiento podría ser diferente. La extensión por defecto no maneja por si mismo `allow_access` y `allow_publish`, se usa un soporte interno en caso que la extensión no este lista para ella.

Para mas información sobre permisos, visite [la sección de autenticación](auth.md).

### Uso

```yalm
packages:
  # scoped packages
  '@scope/*':
    access: all
    publish: all
    proxy: server2

  'private-*':
    access: all
    publish: all
    proxy: uplink1

  '**':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    access: all
    publish: all
    proxy: uplink2
```

si ninguno esta especificado, por defecto uno se define

```yaml
packages:
  '**':
     access: all
     publish: $authenticated
```

La lista de grupos validos de acuerdo a la extensión por defecto son

```js
'$all', '$anonymous', '@all', '@anonymous', 'all', 'undefined', 'anonymous'
```

Todos los usuarios reciben todo ese grupo de permisos independientemente si es anónimo o no más el grupo proveído por la extensión, en caso de ` htpasswd` se regresa el nombre de usuario por grupo. Por ejemplo, si has iniciado sesión como ` npmUser` el listado de grupos será.

```js
// groups without '$' are going to be deprecated eventually
'$all', '$anonymous', '@all', '@anonymous', 'all', 'undefined', 'anonymous', 'npmUser'
```

Si quieres proteger un grupo específico de paquetes bajo un grupo, necesitas hacer algo así. Vamos a usar un `Regex` que cubre los todos los páquetes prefijos con`npmuser-`. Recomendamos usar un prefijo en sus paquetes, en esa manera es mucho mas sencillo protegerlos.

```yaml
packages:
  'npmuser-*':
     access: npmuser
     publish: npmuser
```

Reinicia `verdaccio` en tu terminal trata de instalar `npmuser-core`.

```bash
$ npm install npmuser-core
npm install npmuser-core
npm ERR! code E403
npm ERR! 403 Forbidden: npmuser-core@latest

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/user/.npm/_logs/2017-07-02T12_20_14_834Z-debug.log
```

Puedes cambiar el comportamiento por defecto usando una diferente extensión de autenticación. `verdaccio` solo revisa si el usuario trata de acceder a un publicar un paquete específico pertenece al grupo correcto.

#### Definir múltiples grupos

Define múltiples grupos de acceso es sencillo, solo defínelos con espacios entre ellos.

```yaml
  'company-*':
    access: admin internal
    publish: admin
    proxy: server1
  'supersecret-*':
    access: secret super-secret-area ultra-secret-area
    publish: secret ultra-secret-area
    proxy: server1

```

#### Bloqueando el acceso a paquetes

Si quieres bloquear el acceso/publicar a un grupo de paquetes. Solo, no definas `access` y `publish`.

```yaml
packages:
  'old-*':
  '**':
     access: all
     publish: $authenticated
```

#### Bloqueando proxy a un grupo específico de paquetes

Podrías querer bloquear uno o varios paquetes que sean descargados desde repositorios remotos, pero al mismo tiempo, permitir otros acceder a diferentes * uplinks*.

Veamos el siguiente ejemplo:

```yaml
packages:
  'jquery':
     access: $all
     publish: $all
  'my-company-*':
     access: $all
     publish: $authenticated     
  '**':
     access: all
     publish: $authenticated
     proxy: npmjs         
```

Let's describe what we want with the example above:

* I want to host my own `jquery` dependency but I need to avoid proxying it.
* I want all dependencies that match with `my-company-*` but I need to avoid proxying them.
* I want to proxying all the rest dependencies.

Be **aware that the order of your packages definitions is important and always use double wilcard**. Because if you do not include it `verdaccio` will include it for you and the way how your dependencies are solved will be affected.

### Configuración

You can define mutiple `packages` and each of them must have an unique `Regex`.

| Propiedad | Tipo    | Requerido | Ejemplo        | Soporte | Descripción                                                |
| --------- | ------- | --------- | -------------- | ------- | ---------------------------------------------------------- |
| access    | string  | No        | $all           | all     | define que grupos estan permitidos para acceder al paquete |
| publish   | string  | No        | $authenticated | all     | defini que grupos estan permitidos a publicar              |
| proxy     | string  | No        | npmjs          | all     | limita las busquedas a un uplink específico                |
| storage   | boolean | No        | [true,false]   | all     | TODO                                                       |

> We higlight recommend do not use **allow_access**/**allow_publish** and **proxy_access** anymore, those are deprecated and soon will be removed, please use the short version of each of those (**access**/**publish**/**proxy**).
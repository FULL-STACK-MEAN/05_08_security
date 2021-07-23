# Control de accesos basado en roles en MongoDB

## Conceptos tenemos en sistema RBAC

- Roles. Agrupación lógica de conjunto de privilegios que son asignados a los usuarios.

- Privilegios. Especificación del permiso para realizar operaciones sobre los recursos.

- Recursos. Componentes del sistema sobre los que se pueden realizar operaciones (colecciones, 
bases de datos, usuarios, índices, etc).

- Acciones. Tipos de operaciones que podemos llevar a cabo sobre los recursos (lectura, escritura,
backup, etc...)

## Roles propios de MongoDB

### Roles de usuario de base de datos

read Realizar operaciones de lectura en las colecciones de una base de datos

Login con user "Sara" (que tiene permisos para crear usuarios)

use app4

db.createUser({
    user: "Carlos",
    pwd: "carlos1234",
    roles: ["read"]
})

Login en otra shell para carlos

mongo --authenticationDatabase "app4" -u "Carlos"

use app4

show collections

db.budgets.find() // ok

db.budgets.insert({a: 1})  // Error porque solo tiene permisos de lecturas
WriteCommandError({
        "ok" : 0,
        "errmsg" : "not authorized on app4 to execute command { insert: \"budgets\", ordered: true, lsid: { id: UUID(\"dc2d8812-96e9-4c9e-89df-66576b33023c\") }, $db: \"app4\" }",
        "code" : 13,
        "codeName" : "Unauthorized"
})

readWrite Permisos de lectura y escritura en todas las colecciones de la base de datos

Podemos modificar los roles de un usuario con runCommand de la siguiente manera

loguearnos con sara

use app4

db.runCommand({
    udpateUser: "carlos",
    roles: ["readWrite"]
})

Logueamos de nuevo a Carlos

> db.budgets.insert({a: 1})
WriteResult({ "nInserted" : 1 }) //ok

### Se puede extender un usuario a varias bases de datos

loguearnos con sara

db.runCommand({
    updateUser: "Carlos",
    roles: [
        {role: "readWrite", db: "app4"},
        {role: "read", db: "marathon"}
    ]
})

### Roles de usuario de administración del sistema

1.- Administración a nivel de una base de datos (en la que el usuario esté creado)

dbAdmin Igual que un readWrite más operaciones sobre el system.profile (logs de monitorización)

userAdmin Permite es administrar los usuarios de esa base de datos

dbAdmin Puede desarrollar operaciones de administración de la base de datos

dbOwner Puede desarrolar todas las operaciones posibles de la base de datos

2.- Administración de todas las bases de datos (estos roles solo se podrán asignar a
usuarios creados en la base de datos admin)

readAnyDatabase

readAndWriteAnyDatabase

userAdminAnyDatabase // administrar los usuarios de todas las bases de datos (incluyendo admin)

dbAdminAnyDatabase // realizar operaciones de administración en todas las bases de datos

clusterMonitor // Acceso a métodos de monitorización de los cluster 

clusterManager // Administrar los cluster

clusterAdmin // idem a manager + borrado de bases de datos en cluster

backup // permite realizar operaciones de backup

restore // idem restore

3.- "Superusers" (solo pueden ser asignados a usuarios creados en la base de datos admin)

dbOwner // sobre la base de datos admin 

root // idem anterior más clusterAdmin más backup más restore *máximo nivel de permisos



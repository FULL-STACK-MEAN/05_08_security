# Roles propios en MongoDB

Podemos crear roles propios con unos determinados privilegios para combinación
de recurso/acciones y asignarlos a los usuarios que necesitemos.

Se establaece también a nivel de base de datos

db.createRole({
    role: <nombre-del-rol>,
    privileges: [
        {resource: <recurso>, actions: [<accion1>, <accion2>, ...]},
        ...
    ],
    roles: [
        <otros-roles>
    ]
})

resource: {db: <base-datos>, collection: <colección>}
actions: ["find","insert","update",...]

acciones en https://docs.mongodb.com/manual/reference/privilege-actions/index.html

use marathon

db.createRole({
    role: "leerTodasColeccionesEscribirParticipantes",
    privileges: [
        {
            resource: {db: "marathon", collection: "participantes"},
            actions: ["find","insert","update"]
        }
    ],
    roles: [
        {role: "read", db: "marathon"}
    ]
})

Lo asignamos a un nuevo usuario

db.createUser({
    user: "Lucia",
    pwd: "lucia1234",
    roles: ["leerTodasColeccionesEscribirParticipantes"]
})
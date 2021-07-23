# Autenticación en MongoDB

## Mecanismos de autenticación.

MongoDB utiliza varios mecanismos de autenticación en el sistema:

Community server

- SCRAM por defecto

- x.509 Certicate Authentication

Enterprise

- LDSP

- Kerberos

En una implementación de mongo (salvo Atlas), por defecto los servidores (mongod ó mongos) se levantan
sin autenticación. 

## Creación de usuarios.

createUser() los usuarios se definen a nivel de base de datos

Sintaxis

db.createUser({
    user: <nombre>,
    pwd: <contraseña>
    customData: <datos-adicionales-del-usuario>, // opcional
    roles: [
        {<doc-opciones-rol>| <rol> },
        ...
    ]
})

Práctica

Levantamos mongod sin seguridad y vamos a crear un usuario

use marathon

db.createUser({
    user: "Juan",
    pwd: "juan1234",
    customData: {departamento: "IT", dni: "46589832T"},
    roles: [
        "readWrite"
    ]
})

El usuario se establece a nivel de base de datos y sus permisos se determinan para
esa base de datos, pero los valores propios de ese usuario estarán en una colección
común para todas las bases de datos system.users que está en la base de datos admin.

Entonces, ¿como creamos un usuario que tenga acceso y permisos para todas las bases de datos?

Haciendo lo mismo pero para la base de datos admin

use admin

db.createUser({
    user: "Sara",
    pwd: "sara1234",
    customData: {departamento: "IT", dni: "354736545E"},
    roles: [
        "userAdminAnyDatabase"
    ]
})

Podemos visualizar los usuarios accediendo a esa colección

use admin
db.system.users.find()
{
        "_id" : "marathon.Juan",
        "userId" : UUID("fbd2526a-2639-4577-858b-74e4a356c003"),
        "user" : "Juan",
        "db" : "marathon",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "Ngpzc0TN+rQu0DLbhNqPCg==",
                        "storedKey" : "8X3Sy58pnRmS57555MgWicXQTjs=",
                        "serverKey" : "nE0hTjQEyYgy247v5qpKrdMqpNw="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "+QcnG/QAJi0j7Zbqur2GZCh1RyYeN2LKNjszgg==",
                        "storedKey" : "ReYgGUwJEWjzWxd2wo90Zx6NefF+VGr9YH/B8jMyFuU=",
                        "serverKey" : "VGhe4MxhbIfSSBQNTv6YIxVqmD+SDU3YjJXwlyd4LLw="
                }
        },
        "customData" : {
                "departamento" : "IT",
                "dni" : "46589832T"
        },
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "marathon"
                }
        ]
}
{
        "_id" : "admin.Sara",
        "userId" : UUID("c28833d7-0e43-40d9-a6bd-3cf2a01910c0"),
        "user" : "Sara",
        "db" : "admin",
        "credentials" : {
                "SCRAM-SHA-1" : {
                        "iterationCount" : 10000,
                        "salt" : "9ln8aioCHzxti0q5lw6cbw==",
                        "storedKey" : "l3VtpByJ+DuU95TbewXuPfoSALE=",
                        "serverKey" : "PpIV2Oahh6iFzakC/apn6O0qC0Q="
                },
                "SCRAM-SHA-256" : {
                        "iterationCount" : 15000,
                        "salt" : "JBTMt4ad+ZucODsqo0PtRClyQ9UMVOR6ZG5GIQ==",
                        "storedKey" : "pM4W6ByeTyvdX2DPliCBXisml+6SS9IfbCa0LtniCmY=",
                        "serverKey" : "S6VTBL06mibHb+CKwiS9+WtrIZ7+kw7g7fjRT2rcmJw="
                }
        },
        "customData" : {
                "departamento" : "IT",
                "dni" : "354736545E"
        },
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ]
}

## Despliegue o levantado de servidores en modo seguro

a mongod ó mongos añadir el flag --auth

mongod --dbpath <ruta-directorio-archivos> --port <puerto> --auth

## Conexión mediante shell mongo a un servidor en modo seguro

1.- mongo --authenticationDatabase "marathon" -u "Juan" -p "juan1234"

2.- mongo --authenticationDatabase "marathon" -u "Juan" // y contraseña en el port

3.- mongo // y usar desde la base de datos el método db.auth(<usuario>,<contraseña>)

Si accedemos con un usuario que esté creado en la base datos admin tendremos
acceso a todas las base de datos

mongo --authenticationDatabase "admin" -u "Sara"

## Cambio de contraseña

Desde la base de datos donde esté el usuario asignado con los permisos necesarios (ver roles)

use marathon

db.runCommand({
    updateUser: "Juan",
    pwd: "juan4321"
})

¿Se puede el propio usuario cambiar su propia contraseña?

Si, pero deberá tener el role changeOwnPasswDataRole

## Eliminación de usuario

Elíminar el registro desde la colección system.users

## Particularidad en la creación de usuarios en MongoDB

Como los usuarios se crean a nivel de base de datos podríamos tener un mismo nombre
de usuario en diferentes bases de datos. No sería buena práctica porque puede inducir
a error, sería mejor extender el usuario con los roles necesarios en varias bases de
datos (ver 02_rbac.md)

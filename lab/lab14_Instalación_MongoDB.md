# LABORATORIO 14 NoSQL 

## Instalación de MongoDB.

Instalamos mongodb en docker, para ello abrimos la termina de docker desktop y ejecutamos el siguiente codigo: 

```
docker run -d --name mongodb-crud  -p 27017:27017  -e MONGO_INITDB_ROOT_USERNAME=admin   -e MONGO_INITDB_ROOT_PASSWORD=admin123   mongo:latest
```


Ahora entramos al contenedor de docker:
```
docker exec -it mongodb-crud bash
```

Si necesitas autenticación (porque creaste usuario root):
```
mongosh -u admin -p admin123
```
## Verificar que MongoDB está funcionando

Dentro de mongosh, prueba:
```
show dbs
```

Deberías ver algo como:

admin
config
local

## Crear una base y colección de prueba

Crear la BD prueba
```
use prueba
```
Insertar datos:
```
db.alumnos.insertOne({ nombre: "Juan", edad: 22 })
```

```
db.alumnos.find()
```
Si ves el documento, MongoDB está funcionando perfecto.



## Descargar MongoDB Compass (Windows)

Entra al sitio oficial:

👉 https://www.mongodb.com/try/download/compass

Selecciona:

Platform: Windows 64-bit (o MSI Installer)

Package: MSI

Descarga y ejecuta el instalador.

Instálalo con las opciones por defecto.


La URI de conexión será:
```
mongodb://admin:admin123@localhost:27017/?authSource=admin
```

authSource=admin es necesario porque el usuario root se crea en la base admin.
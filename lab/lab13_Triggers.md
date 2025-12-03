# Laboratorio 12: Funciones

## 🎯 1. Objetivo del laboratorio

Al finalizar el laboratorio, el estudiante será capaz de:

Explicar qué es un Trigger en MySQL.

Crear un trigger sencillo que se ejecute automáticamente al insertar datos.

Entender las palabras clave BEFORE, AFTER, INSERT, UPDATE, DELETE.

Ver cómo un trigger puede almacenar información en una tabla "histórica" o "log".

## 🗃 2. Script de base de datos de ejemplo

Usaremos una base simple con dos tablas:

alumnos

log_alumnos (para registrar automáticamente las inserciones)

Ejecutar este script en MySQL:
```
DROP DATABASE IF EXISTS lab_triggers;
CREATE DATABASE lab_triggers;
USE lab_triggers;

-- Tabla principal
CREATE TABLE alumnos (
    id_alumno INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100)
);

-- Tabla para guardar eventos
CREATE TABLE log_alumnos (
    id_log INT AUTO_INCREMENT PRIMARY KEY,
    id_alumno INT,
    nombre VARCHAR(100),
    fecha_registro DATETIME
);
```

## 📘 3. Explicación simple del ejemplo

Un trigger es un bloque de código que MySQL ejecuta automáticamente cuando ocurre un evento en una tabla, como:

INSERT

UPDATE

DELETE

Los triggers se usan para:

Guardar auditorías (historial de cambios).

Validar información.

Actualizar tablas relacionadas.

Evitar inconsistencias.

En este laboratorio, veremos un caso típico:

Cuando se inserta un alumno en la tabla alumnos, automáticamente se inserta un registro en log_alumnos con la fecha y nombre.

Esto permite a los alumnos entender la utilidad de los triggers:

Automatización

Auditoría

Integridad de datos

## 🔧 4. Script del TRIGGER de ejemplo

Crearemos un trigger AFTER INSERT.
```
USE lab_triggers;


CREATE TRIGGER trg_alumno_insert
AFTER INSERT ON alumnos
FOR EACH ROW
BEGIN
    INSERT INTO log_alumnos (id_alumno, nombre, fecha_registro)
    VALUES (NEW.id_alumno, NEW.nombre, NOW());
END

```

## 📝 5. Explicación del trigger (para la clase)

AFTER INSERT
→ El trigger se ejecuta después de insertar un nuevo alumno.

FOR EACH ROW
→ Se ejecuta una vez por cada fila insertada.

NEW.id_alumno y NEW.nombre
→ Accedemos a los valores que se acaban de insertar.

NOW()
→ Guarda la fecha y hora actual.

Acción del trigger:
→ Insertar automáticamente un registro en log_alumnos.

## ▶️ 6. Probar el trigger

Insertar alumnos:
```
INSERT INTO alumnos (nombre, email)
VALUES ('Carlos Ruiz', 'carlos@example.com'),
       ('Lucía Ramos', 'lucia@example.com');
```

Consultar el log:
```
SELECT * FROM log_alumnos;
```

Resultado esperado:

id_log	id_alumno	nombre	fecha_registro
1	1	Carlos Ruiz	2024-XX-XX XX:XX:XX
2	2	Lucía Ramos	2024-XX-XX XX:XX:XX

## 🧩 7. Ejercicios sugeridos
Ejercicio 1 – Trigger BEFORE UPDATE

Crear un trigger que guarde en una tabla log_cambios_email el email anterior cada vez que se actualiza el email de un alumno.

Ejercicio 2 – Trigger AFTER DELETE

Crear un trigger que guarde en un log qué alumno fue eliminado y cuándo.

Ejercicio 3 – Validación con BEFORE INSERT

Evitar insertar un email duplicado usando un trigger que valide antes de insertar.

## 📊 8. Tabla comparativa: TRIGGER vs PROCEDURE vs FUNCTION
| Característica           | TRIGGER                                                | PROCEDURE                                           | FUNCTION                                |
|--------------------------|--------------------------------------------------------|------------------------------------------------------|------------------------------------------|
| Se ejecuta automáticamente | ✔ Sí, lo activa un evento (INSERT/UPDATE/DELETE)       | ✖ No, debe llamarse con `CALL`                      | ✖ No, se usa dentro de un `SELECT`       |
| Se llama manualmente     | ✖ No                                                   | ✔ Sí                                                 | ✔ Sí, como parte de una expresión        |
| Maneja múltiples filas    | ✔ Sí, se ejecuta por cada fila afectada               | ✔ Sí                                                 | ✖ No, devuelve solo un valor             |
| Uso típico               | Auditoría, validaciones, sincronización de datos       | Procesos complejos, tareas largas                    | Cálculos, transformaciones, validaciones |

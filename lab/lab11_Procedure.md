# Laboratorio 11: Stored Procedures and Cursores

## 🧪 1. Objetivo del laboratorio

Al finalizar el laboratorio, el estudiante será capaz de:

Crear una base de datos de ejemplo en MySQL.

Entender el concepto básico de un stored procedure (procedimiento almacenado).

Crear un stored procedure sencillo con parámetros de entrada.

Ejecutar el stored procedure y analizar el resultado.

## 🗂 2. Script para crear y cargar la base de datos de ejemplo

Este ejemplo usará una base de datos académica muy simple:

Tabla alumnos

Tabla cursos

Tabla notas (relaciona alumnos con cursos y sus notas)

💡 Copia y ejecuta este script completo en MySQL (Workbench, CLI, DBeaver, etc.):
```
-- 1. Crear la base de datos
DROP DATABASE IF EXISTS lab_procedures;
CREATE DATABASE lab_procedures;
USE lab_procedures;

-- 2. Crear tabla ALUMNOS
CREATE TABLE alumnos (
    id_alumno INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100)
);

-- 3. Crear tabla CURSOS
CREATE TABLE cursos (
    id_curso INT AUTO_INCREMENT PRIMARY KEY,
    nombre_curso VARCHAR(100) NOT NULL
);

-- 4. Crear tabla NOTAS
CREATE TABLE notas (
    id_nota INT AUTO_INCREMENT PRIMARY KEY,
    id_alumno INT NOT NULL,
    id_curso INT NOT NULL,
    nota DECIMAL(4,2) NOT NULL,
    FOREIGN KEY (id_alumno) REFERENCES alumnos(id_alumno),
    FOREIGN KEY (id_curso) REFERENCES cursos(id_curso)
);

-- 5. Insertar datos de ejemplo en ALUMNOS
INSERT INTO alumnos (nombre, email) VALUES
('Ana Pérez', 'ana@example.com'),
('Juan López', 'juan@example.com'),
('María García', 'maria@example.com');

-- 6. Insertar datos de ejemplo en CURSOS
INSERT INTO cursos (nombre_curso) VALUES
('Base de Datos'),
('Programación I'),
('Matemática');

-- 7. Insertar datos de ejemplo en NOTAS
-- Ana
INSERT INTO notas (id_alumno, id_curso, nota) VALUES
(1, 1, 15.5),
(1, 2, 18.0),
(1, 3, 14.0);

-- Juan
INSERT INTO notas (id_alumno, id_curso, nota) VALUES
(2, 1, 12.0),
(2, 2, 13.5),
(2, 3, 11.0);

-- María
INSERT INTO notas (id_alumno, id_curso, nota) VALUES
(3, 1, 19.0),
(3, 2, 17.5),
(3, 3, 18.0);
```

Con esto ya tienes datos suficientes para jugar con consultas y procedures.

## 📘 3. Explicación simple del ejemplo de Stored Procedure

La idea del ejemplo será:

👉 Crear un stored procedure que reciba el id_alumno como parámetro y devuelva el promedio de sus notas junto con el nombre del alumno.

Conceptualmente:

Sin procedure, el profesor tendría que escribir este SELECT cada vez que quiera ver el promedio de un alumno.

Con un procedure, encapsulamos esa lógica en un “bloque de código” en la base de datos, y solo llamamos:
```
CALL ObtenerPromedioAlumno(1);
```

Beneficios que puedes explicar en clase:

Reutilización de consultas.

Centralizar lógica en la base de datos.

Menos errores al escribir siempre el mismo SELECT.

Posibilidad de recibir parámetros (como el ID de alumno).

## 🧾 4. Script del Stored Procedure de ejemplo

🔧 Procedure: ObtenerPromedioAlumno
Parámetro de entrada: p_id_alumno (INT)
Salida: un SELECT con nombre del alumno y su promedio.

4.1. Crear el Stored Procedure

💡 Importante: cambiar el delimitador antes y después (DELIMITER //), porque el procedure tiene varios puntos y coma internos.
```
USE lab_procedures;

DELIMITER //

CREATE PROCEDURE ObtenerPromedioAlumno(IN p_id_alumno INT)
BEGIN
    SELECT 
        a.id_alumno,
        a.nombre,
        AVG(n.nota) AS promedio_notas
    FROM alumnos a
    INNER JOIN notas n ON a.id_alumno = n.id_alumno
    WHERE a.id_alumno = p_id_alumno
    GROUP BY a.id_alumno, a.nombre;
END
//

DELIMITER ;
```

4.2. Explicación línea por línea (para tus alumnos)

DELIMITER //
Cambia temporalmente el delimitador de comandos de ; a // para que MySQL no corte el procedure a la mitad.

CREATE PROCEDURE ObtenerPromedioAlumno(IN p_id_alumno INT)
Crea un procedure llamado ObtenerPromedioAlumno con un parámetro de entrada (IN) llamado p_id_alumno de tipo entero.

BEGIN ... END
Marca el bloque de instrucciones que forman parte del procedure.

SELECT ...
Hace el cálculo del promedio:

Une alumnos con notas.

Filtra por el alumno cuyo id se pasa por parámetro.

Calcula AVG(n.nota) para obtener el promedio.

Agrupa por id y nombre del alumno.

DELIMITER ;
Vuelve a dejar el delimitador normal ;.

## ▶️ 5. Cómo ejecutar (probar) el Stored Procedure

Puedes hacer pruebas con los IDs de ejemplo:
```
-- Promedio de Ana (id = 1)
CALL ObtenerPromedioAlumno(1);

-- Promedio de Juan (id = 2)
CALL ObtenerPromedioAlumno(2);

-- Promedio de María (id = 3)
CALL ObtenerPromedioAlumno(3);
```

El resultado será algo como:


|id_alumno|	nombre    |	promedio_notas|
|---------|-----------|---------------|
|1        |	Ana Pérez |	15.83         |

(Valores aproximados según tus datos.)

## 🧩 6. Ideas para actividades del laboratorio

Puedes pedir a los estudiantes que:

Modifiquen el procedure para que:

Si el alumno no tiene notas, muestre un mensaje (por ahora pueden ver que no retorna filas).

Creen un nuevo procedure, por ejemplo:

ListarNotasAlumno(p_id_alumno) → que muestre curso y nota por alumno.

Creen un procedure sin parámetros:

ListarPromediosTodos() → que liste todos los alumnos con su promedio.
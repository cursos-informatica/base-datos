# Laboratorio 12: Funciones

## 🧪 1. Objetivo del laboratorio (FUNCTION)

Al finalizar el laboratorio, el estudiante será capaz de:

Entender qué es una función almacenada en MySQL.

Crear una FUNCTION con parámetros de entrada.

Usar la función dentro de un SELECT.

Diferenciar conceptualmente una FUNCTION de un PROCEDURE.

## 🗂 2. Base de datos de ejemplo

Podemos usar la misma BD del laboratorio de procedures (lab_procedures).
Si ya la tienes creada, puedes saltarte este paso.

Si no, aquí está nuevamente el script resumido:
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
INSERT INTO notas (id_alumno, id_curso, nota) VALUES
-- Ana
(1, 1, 15.5),
(1, 2, 18.0),
(1, 3, 14.0),
-- Juan
(2, 1, 12.0),
(2, 2, 13.5),
(2, 3, 11.0),
-- María
(3, 1, 19.0),
(3, 2, 17.5),
(3, 3, 18.0);
```

## 📘 3. Idea simple de la FUNCTION

Vamos a crear una función que, dado el id_alumno, nos devuelva solo el promedio numérico de sus notas.

Ejemplo:
```
SELECT ObtenerPromedioAlumnoFn(1);
```

Y que ese resultado lo podamos usar dentro de una consulta:
```
SELECT 
    a.id_alumno,
    a.nombre,
    ObtenerPromedioAlumnoFn(a.id_alumno) AS promedio
FROM alumnos a;
```

## 🧾 4. Script de la FUNCTION en MySQL

🔧 Nombre de la función: ObtenerPromedioAlumnoFn
🔧 Parámetro de entrada: p_id_alumno INT
🔧 Valor de retorno: DECIMAL(5,2) (promedio de notas)

```
USE lab_procedures;

DELIMITER //

CREATE FUNCTION ObtenerPromedioAlumnoFn(p_id_alumno INT)
RETURNS DECIMAL(5,2)
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE v_promedio DECIMAL(5,2);

    SELECT AVG(n.nota)
    INTO v_promedio
    FROM notas n
    WHERE n.id_alumno = p_id_alumno;

    RETURN v_promedio;
END
//

DELIMITER ;
```

## 🔍 5. Explicación de la FUNCTION (para que la expliques en clase)

CREATE FUNCTION ObtenerPromedioAlumnoFn(p_id_alumno INT)
Declara una función almacenada llamada ObtenerPromedioAlumnoFn con un parámetro de entrada.

RETURNS DECIMAL(5,2)
Indica el tipo de dato que devuelve la función (un número con 5 dígitos, 2 decimales).

DETERMINISTIC
Indica que la función siempre devuelve el mismo resultado para los mismos parámetros (buena práctica cuando solo lee datos).

READS SQL DATA
Indica que la función solo lee datos de tablas (no hace INSERT, UPDATE ni DELETE).

DECLARE v_promedio DECIMAL(5,2);
Variable local donde guardaremos el promedio.

SELECT AVG(n.nota) INTO v_promedio ...
Calcula el promedio y lo guarda en v_promedio.

RETURN v_promedio;
Devuelve el valor de la variable: resultado final de la función.

## ▶️ 6. Cómo probar la FUNCTION
6.1. Llamar la función directamente

```
SELECT ObtenerPromedioAlumnoFn(1) AS promedio_ana;
SELECT ObtenerPromedioAlumnoFn(2) AS promedio_juan;
SELECT ObtenerPromedioAlumnoFn(3) AS promedio_maria;
```

6.2. Usarla dentro de un SELECT con la tabla alumnos

```
SELECT 
    a.id_alumno,
    a.nombre,
    ObtenerPromedioAlumnoFn(a.id_alumno) AS promedio
FROM alumnos a;
```

Esto es muy bueno pedagógicamente porque muestra que una función:

Se comporta como cualquier otra función de MySQL (NOW(), COUNT(), etc.).

Se puede usar en SELECT, WHERE, ORDER BY, etc.

## 🔁 7. Diferencias rápidas: PROCEDURE vs FUNCTION (para comentar al final)

Puedes cerrar el laboratorio con esta tabla:

| Característica        | PROCEDURE                                                                 | FUNCTION                                         |
|-----------------------|----------------------------------------------------------------------------|--------------------------------------------------|
| Devuelve valor único  | No necesariamente (puede devolver 0+ resultados, parámetros OUT, etc.)     | Sí, siempre devuelve un valor                     |
| Se usa en             | `CALL nombre_procedure(...)`                                               | En `SELECT`, `WHERE`, `ORDER BY`, etc.           |
| En consultas SQL      | No se puede usar como expresión                                            | Sí, se usa como una función normal               |
| Uso típico            | Procesos, tareas completas, reportes                                       | Cálculos, transformaciones, validaciones         |


##🧩 8. Ideas de ejercicios para tus estudiantes

Ejercicio 1:
Crear una función CantidadCursosAlumno(p_id_alumno) que devuelva cuántos cursos tiene el alumno en la tabla notas.

Ejercicio 2:
Crear una función EstadoAlumno(p_id_alumno) que:

Devuelva 'APROBADO' si el promedio es ≥ 14

Devuelva 'DESAPROBADO' en caso contrario

(Usando IF dentro de la función.)

Ejercicio 3:
Usar la función en un ORDER BY:
```
SELECT 
    a.id_alumno,
    a.nombre,
    ObtenerPromedioAlumnoFn(a.id_alumno) AS promedio
FROM alumnos a
ORDER BY promedio DESC;
```

# appTrasporteUMG – CRUD del caso de Transporte (db_paqueteria)

Aplicación de escritorio en **Java (Swing) + MySQL** con CRUD completo y validaciones para las tablas
`camion`, `camionero`, `paquete` y `camionero_camion` (más `ciudad` como tabla de apoyo para los paquetes).

## 1. Requisitos

- JDK 19 o superior y NetBeans (el proyecto es Maven).
- MySQL Server + MySQL Workbench.
- Git (para subirlo a GitHub).

## 2. Crear la base de datos

1. Abre MySQL Workbench y conéctate (usuario `root`).
2. `File > Open SQL Script…` → `sql/db_paqueteria.sql` → botón del rayo (ejecutar todo).
3. Comprueba: `USE db_paqueteria; SHOW TABLES;` → deben salir 5 tablas.

> El script usa `CREATE TABLE IF NOT EXISTS`. Si ya tenías tablas con otras columnas, no las modifica.
> En ese caso borra la base (`DROP DATABASE db_paqueteria;`) y vuelve a ejecutar el script.

## 3. Abrir el proyecto en NetBeans

`File > Open Project…` → selecciona la carpeta `appTrasporteUMG` (tiene el icono de Maven).
NetBeans descarga `mysql-connector-j` automáticamente la primera vez (necesita internet).

## 4. Configurar la conexión

Por defecto: `localhost:3306`, base `db_paqueteria`, usuario `root`, contraseña `root`.
Si tu contraseña es otra, cámbiala en `src/main/java/umg/DAO/ConexionDB.java` (valor por defecto de `PASS`)
o ejecuta con `-Ddb.pass=TU_CLAVE`.

## 5. Ejecutar

Clic derecho al proyecto → **Run**. Clase principal: `umg.UI.AppTrasporteUMG`.
Se abre el menú con un botón por tabla.

## 6. Cómo está enlazado todo

```
AppTrasporteUMG (main, prueba la conexión)
   └─ FrmPrincipal (menú)
        ├─ FrmCiudad            ─┐
        ├─ FrmCamionero          │  todos extienden FrmBase
        ├─ FrmCamion             │  (campos + botones + tabla + manejo de errores)
        ├─ FrmPaquete            │
        └─ FrmCamioneroCamion   ─┘
              │ validan con  → Util.Validador  (lanza ValidacionException)
              │ guardan con  → DAO.*DAO        (PreparedStatement, try-with-resources)
              │ usan modelos → Modelo.*        (records)
              └─ DAO.ConexionDB.abrir()  →  MySQL db_paqueteria
```

- **Modelo**: datos de cada tabla (`Camion`, `Camionero`, `Paquete`, `CamioneroCamion`, `Ciudad`).
- **DAO**: un SQL por operación: `insertar`, `listar`, `actualizar`, `eliminar` (+ `existe…` para duplicados).
- **UI**: `FrmBase` implementa los botones Guardar / Actualizar / Eliminar / Limpiar y la tabla;
  cada formulario solo define sus campos y llama a su DAO.
- **Util**: `Validador` (reglas) e `Item` (id + texto para los combos).

## 7. Validaciones implementadas

| Tabla | Validaciones |
|---|---|
| camionero | nombre/apellido/población solo letras; dirección obligatoria; salario numérico > 0 con 2 decimales; DPI 13 dígitos y **único**; teléfono 8 dígitos |
| camion | matrícula formato `C-123ABC` (normaliza a mayúsculas) y **única**; modelo obligatorio; tipo de lista; potencia entera 50–1500 CV |
| paquete | descripción, destinatario (solo letras) y dirección obligatorios; ciudad y camionero obligatorios (combos con datos reales) |
| camionero_camion | camionero y camión obligatorios; **no se repite la pareja** (clave primaria compuesta); fecha `AAAA-MM-DD` válida y no futura |
| todas | no se elimina un registro que otra tabla usa (llave foránea) → mensaje claro; confirmación antes de eliminar; seleccionar fila antes de actualizar/eliminar |

## 8. Orden recomendado para la demostración y las capturas

1. **Ciudades**: crear 2-3 (ya hay 5 de ejemplo).
2. **Camioneros**: crear uno nuevo, actualizar el salario, intentar DPI de 12 dígitos (validación), intentar repetir DPI.
3. **Camiones**: crear `P-777XYZ`, intentar matrícula `123` (validación), repetir matrícula.
4. **Paquetes**: crear uno con ciudad y camionero; intentar guardar sin elegir ciudad.
5. **Asignaciones**: asignar camionero↔camión; repetir la misma pareja (validación).
6. **Eliminar** en orden inverso: asignación → paquete → camión/camionero. Intentar eliminar un camionero con paquetes para mostrar la protección.

## 9. Consultas para las capturas de la base de datos (Workbench)

```sql
USE db_paqueteria;
SELECT * FROM camion;
SELECT * FROM camionero;
SELECT * FROM paquete;
SELECT * FROM camionero_camion;
```

## 10. Subir a GitHub

```bash
cd appTrasporteUMG
git init
git add .
git commit -m "CRUD caso Transporte: camion, camionero, paquete y camionero_camion"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/appTrasporteUMG.git
git push -u origin main
```

(Antes crea el repositorio vacío en github.com → *New repository* → nombre `appTrasporteUMG`, sin README.)

## 11. Preguntas probables del docente

- **¿Qué es un DAO?** Clase que concentra el acceso a la base de datos de una tabla, separando el SQL de la interfaz.
- **¿Por qué `PreparedStatement`?** Evita inyección SQL y maneja bien tipos y caracteres especiales.
- **¿Cómo evitan duplicados?** Doble capa: el `Validador`/DAO consulta antes (`existeDpi`, `existeMatricula`) y la base tiene `UNIQUE`.
- **¿Qué pasa al borrar un camionero con paquetes?** La llave foránea (`ON DELETE RESTRICT`) lo impide; la app captura el error 1451 y muestra un mensaje.
- **¿Cómo funciona `camionero_camion`?** Tabla intermedia N:M con clave primaria compuesta (`idCamionero`, `idCamion`).
- **¿Para qué sirve `FrmBase`?** Evita repetir código: el CRUD visual es igual en todas las tablas.

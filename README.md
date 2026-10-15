# Oracle Cold Backup

Herramienta de línea de comandos en **Python** que automatiza el **respaldo y la restauración en frío** de una base de datos **Oracle (CDB/PDB)**.

## El problema

Respaldar Oracle a mano implica ubicar decenas de archivos (datafiles, controlfiles, redo logs y archive logs) repartidos en varios discos, copiarlos uno por uno y, al restaurar, devolver cada uno a su ruta exacta. Es lento y es fácil equivocarse.

## La propuesta

El script hace todo el proceso en unos cuantos pasos guiados:

1. **Indica la consulta SQL** que hay que ejecutar para listar los archivos de la base de datos.
2. **Lee la salida de SQL\*Plus** y extrae las rutas con expresiones regulares (incluso si vienen cortadas en varias líneas).
3. **Genera una estructura de respaldo ordenada** que conserva el disco de origen, el CDB, el PDB y el tipo de archivo.
4. **Muestra un resumen y pide confirmación** antes de copiar nada.
5. **Copia los archivos** conservando sus metadatos y **registra solo los respaldos completos y sin errores**.
6. **Restaura**: reconstruye la ruta original de cada archivo a partir de la estructura del respaldo y lo regresa a su lugar. No deja restaurar si no hay un respaldo completo registrado.

```
/u01/app/oracle/oradata/ORCL/datafile/system01.dbf   →   /u03/u01/ORCL/datafile/system01.dbf
/u01/app/oracle/fast_recovery_area/ORCL/onlinelog/…  →   /u03/u01/fast_recovery_area/ORCL/onlinelog/…
/u02/ORCL/controlfile/control03.ctl                  →   /u03/u02/ORCL/controlfile/control03.ctl
```

## Tecnologías

- **Python 3.13** · `pathlib`, `shutil`, `re`, `os`
- **[Rich](https://github.com/Textualize/rich)** para la interfaz de terminal (tablas, colores, indicadores de progreso)
- **[uv](https://github.com/astral-sh/uv)** para gestionar dependencias
- **Oracle Database** (arquitectura multitenant, Fast Recovery Area)

## Qué busco lograr

- Automatizar una tarea real de administración de bases de datos (DBA).
- Procesar texto con expresiones regulares y manipular rutas de archivos.
- Validaciones de seguridad (confirmación del usuario, bitácora de respaldos y conteo de errores) para evitar dejar la base de datos en un estado inconsistente.
- Una interfaz de terminal clara y fácil de usar.

## Uso

```bash
uv sync
uv run main.py
```

> Se ejecuta en el servidor de Oracle con la base de datos **apagada** (respaldo en frío) y con permisos sobre `/u01`, `/u02` y `/u03`.

## Estado

✅ Respaldo completo · ✅ Restauración completa · 🚧 Respaldo y restauración de archivos individuales


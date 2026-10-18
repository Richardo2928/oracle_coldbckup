# Oracle Backup & Restore CLI

Herramienta de línea de comandos en **Python** que automatiza el **respaldo y la restauración a nivel de sistema operativo** de una base de datos **Oracle multitenant (CDB/PDB)**, tanto en frío como en caliente.

> Proyecto desarrollado para la materia *Administración de Bases de Datos* (I5902). En lugar de copiar a mano decenas de archivos en cada práctica, decidí automatizar el proceso.

---

## El problema

Respaldar Oracle a mano implica ubicar decenas de archivos (`datafiles`, `controlfiles`, `redo logs` y `archived logs`) repartidos en varios discos, copiarlos uno por uno y, al restaurar, devolver cada uno a su ruta exacta. Si se omite uno solo, la base de datos puede quedar inconsistente.

## La solución

El script guía todo el proceso en unos cuantos pasos:

1. **Indica la consulta SQL** que hay que ejecutar según la práctica (respaldo en frío o en caliente).
2. **Lee la salida de SQL\*Plus** y extrae las rutas con expresiones regulares, aunque vengan cortadas en varias líneas.
3. **Genera una estructura de respaldo ordenada** que conserva el disco de origen, el CDB, el PDB y el tipo de archivo.
4. **Muestra un resumen y pide confirmación** antes de copiar nada.
5. **Copia los archivos** conservando sus metadatos y **registra solo los respaldos completos y sin errores**.
6. **Restaura**: reconstruye la ruta original de cada archivo y lo regresa a su lugar. No permite restaurar si no hay un respaldo completo registrado.

```
/u01/app/oracle/oradata/ORCL/datafile/system01.dbf   →   /u03/u01/ORCL/datafile/system01.dbf
/u01/app/oracle/fast_recovery_area/ORCL/onlinelog/…  →   /u03/u01/fast_recovery_area/ORCL/onlinelog/…
/u02/ORCL/controlfile/control03.ctl                  →   /u03/u02/ORCL/controlfile/control03.ctl
```

---

## Aplicación en las prácticas

| Práctica | Cómo ayuda el script |
| :--- | :--- |
| **Práctica 7 – Respaldo en frío** | Copia `datafiles`, `controlfiles` y `redo logs` con la BD apagada y los restaura a su ruta original. |
| **Práctica 10 – Respaldo en caliente** | Incluye los `archived logs` en el respaldo (con la BD en `BEGIN BACKUP`). |
| **Práctica 10 – Point-in-Time Recovery** | Restaura todos los archivos a su ubicación original para levantar la BD en `MOUNT` y ejecutar `RECOVER DATABASE UNTIL TIME`. |

---

## Tecnologías

- **Python 3.13** · `pathlib`, `shutil`, `re`, `os`
- **[Rich](https://github.com/Textualize/rich)** para la interfaz de terminal (tablas, colores, indicadores de progreso)
- **[uv](https://github.com/astral-sh/uv)** para gestionar dependencias
- **Oracle Database** · arquitectura multitenant y Fast Recovery Area

## Aprendizajes clave

- **Parseo con regex:** SQL\*Plus corta las rutas largas en varias líneas; tuve que diseñar un patrón que las reconstruyera completas.
- **El último `archived log`:** `ALTER SYSTEM ARCHIVE LOG CURRENT` genera un archivo nuevo que no aparece en la consulta inicial, así que hay que volver a consultarlo y copiarlo antes de cerrar el respaldo.
- **`RECOVER ... UNTIL TIME` no "viaja en el tiempo":** aplica cambios sobre archivos viejos, por lo que es indispensable restaurar exactamente los archivos del respaldo.
- **`NLS_DATE_FORMAT`:** sin configurarlo en la sesión, Oracle no interpreta bien la fecha de `UNTIL TIME` y no aplica el último archived log.

---

## Uso

```bash
uv sync
uv run main.py
```

> Se ejecuta en el servidor de Oracle con permisos sobre `/u01`, `/u02` y `/u03`. Para el respaldo en caliente, la BD debe estar en modo `BACKUP`; para restaurar, debe estar apagada (`SHUTDOWN IMMEDIATE`).

## Estado

✅ Respaldo completo · ✅ Restauración completa · 🚧 Respaldo y restauración de archivos individuales

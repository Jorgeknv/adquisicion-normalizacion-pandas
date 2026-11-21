# Adquisición y normalización con pandas

**Jorge Navarrete Villanueva · Data Science II**

Este proyecto integra las transacciones de ventas de un CSV con el catálogo de productos de un Excel. El resultado es un Parquet con una fila por transacción validada y el importe de cada venta en soles (PEN). Los datos son ficticios e incluyen casos de calidad para demostrar el tratamiento de nulos y claves sin coincidencia.

## Archivos

| Archivo | Contenido |
| --- | --- |
| [`adquisicion_normalizacion.ipynb`](adquisicion_normalizacion.ipynb) | Notebook ejecutado con la carga, limpieza, integración, cálculo y verificación de Parquet. |
| [`datos/transacciones.csv`](datos/transacciones.csv) | ID de transacción, ID de producto, cantidad y fecha. |
| [`datos/catalogo_productos.xlsx`](datos/catalogo_productos.xlsx) | ID de producto, nombre y precio unitario en la hoja `catalogo`. |
| [`salida/ventas_normalizadas.parquet`](salida/ventas_normalizadas.parquet) | Resultado consolidado listo para leer con pandas o un motor compatible con Parquet. |
| [`requirements.txt`](requirements.txt) | Versiones de las dependencias para reproducir el notebook. |

## Reproducir en VS Code

Con Python 3.12 instalado, abre la carpeta principal del repositorio en VS Code. En una terminal de PowerShell, crea el entorno e instala las dependencias.

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Abre `adquisicion_normalizacion.ipynb`. En **Select Kernel → Python Environments**, elige la `.venv` del proyecto y pulsa **Restart** y **Run All**. El notebook usa rutas relativas a la carpeta principal y regenera `salida/ventas_normalizadas.parquet`. `pandas` transforma los datos, `openpyxl` lee el Excel, `pyarrow` escribe y lee el Parquet e `ipykernel` permite ejecutar las celdas.

También puedes reproducirlo desde la terminal, en la carpeta principal del repositorio, con la ejecución de `nbconvert`.

```powershell
.\.venv\Scripts\python.exe -m nbconvert --to notebook --execute --inplace --ExecutePreprocessor.kernel_name=python3 adquisicion_normalizacion.ipynb
```

En macOS o Linux, usa `python3.12 -m venv .venv` y sustituye `.\.venv\Scripts\python.exe` por `./.venv/bin/python` en los comandos de instalación y ejecución.

## Limpieza y resultado

Los ID se convierten al mismo tipo entero en las dos fuentes y `fecha` se convierte a `datetime`. La unión izquierda usa `id_producto` con `validate="many_to_one"` para verificar que una venta se vincule como máximo a una ficha del catálogo. La columna `total_venta` resulta de multiplicar `cantidad` por `precio_unitario` y redondear a dos decimales por transacción.

Las ventas con ID, cantidad o fecha faltante o inválida se excluyen con un motivo visible. También se separan las ventas sin ficha de producto o con catálogo incompleto. Esa decisión evita inventar cantidades, fechas o precios que afectarían el consolidado.

| Verificación | Resultado esperado con los datos incluidos |
| --- | ---: |
| Transacciones de origen | 30 |
| Transacciones aptas para el cruce | 26 |
| Transacciones excluidas | 6 |
| Transacciones en el Parquet | 24 |
| Unidades vendidas | 58 |
| Importe consolidado en PEN | 3 565,90 |
| Nulos en las columnas finales | 0 |

El notebook muestra las seis transacciones excluidas y concilia `30 = 24 + 6`. La última celda reabre el Parquet y compara sus valores y tipos con la tabla en memoria. Como comprobación puntual, `V001` tiene dos teclados de `79,90` y un `total_venta` de `159,80`.

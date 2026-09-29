# Módulos y Funciones

## `app.py`

**Propósito:** API web opcional que permite ejecutar el pipeline desde un navegador,
sin necesidad de la terminal.

**Cómo ejecutar:**
```bash
uvicorn app:app --reload        # desarrollo
uvicorn app:app --host 0.0.0.0  # producción
```

**Endpoints principales:**

| Método | Ruta              | Descripción                                      |
|--------|-------------------|--------------------------------------------------|
| GET    | `/`               | Interfaz web (formulario)                        |
| POST   | `/upload`         | Sube el Excel semanal a `data/01_raw/`           |
| POST   | `/run`            | Ejecuta `kedro run` con streaming de logs        |
| GET    | `/status`         | `{"running": true/false}`                        |
| GET    | `/outputs`        | Lista los boletines en `data/08_reporting/`      |
| GET    | `/download/{f}`   | Descarga un boletín por nombre                   |

Todos los endpoints requieren autenticación HTTP Basic (`SIPSA_USER` / `SIPSA_PASS`).

---

## `src/sipsa/settings.py`

**Propósito:** Carga el archivo `.env` al inicio del proyecto y registra el resolver
`env:` en OmegaConf para que `parameters.yml` pueda leer variables de entorno con la
sintaxis `${env:NOMBRE_VARIABLE}`.

**Por qué está aquí y no en `__init__.py`:** Kedro ejecuta `settings.py` antes de
cargar el catálogo y los parámetros, garantizando que las variables ya estén disponibles
cuando OmegaConf las interpola.

---

## `src/sipsa/pipeline_registry.py`

**Propósito:** Registra los 4 pipelines del proyecto y los expone bajo sus nombres
(`ingestion`, `transformation`, `aggregation`, `reporting`). Kedro los descubre
automáticamente a través de este archivo.

---

## Pipeline `ingestion` — `src/sipsa/pipelines/ingestion/nodes.py`

### `leer_entrada(ruta_entrada, archivo_entrada) → pd.DataFrame`

Lee el Excel semanal, valida columnas y estandariza tipos.

**Args:**

| Parámetro        | Tipo  | Descripción                                    |
|------------------|-------|------------------------------------------------|
| `ruta_entrada`   | str   | Directorio del Excel (ej. `"data/01_raw"`)    |
| `archivo_entrada`| str   | Nombre del archivo (ej. `"Listado a 06 mar 26.xlsx"`) |

**Returns:** DataFrame con columnas `Grupo`, `Producto`, `Fuente`, `Min(1)`, `Max(1)`,
`P(1)`, `P(-1)`, `Var(1)`, `Tend` — sin filas vacías, precios como float.

**Raises:** `ValueError` si faltan columnas requeridas en el Excel.

**Ejemplo de llamada desde el catálogo:**
```python
# kedro lo llama internamente; para pruebas manuales:
from sipsa.pipelines.ingestion.nodes import leer_entrada
df = leer_entrada("data/01_raw", "Listado a 06 mar 26.xlsx")
```

---

## Pipeline `transformation` — `src/sipsa/pipelines/transformation/nodes.py`

### `aplicar_mappings(df, mapping_productos, mapping_fuentes, mapping_grupos) → pd.DataFrame`

Añade las columnas `Rproducto`, `RFuente` y `RGrupo` con los códigos numéricos.
Equivale al `DATA Boletin1` del SAS. Descarta filas sin `Rproducto` conocido.

**Args:**

| Parámetro          | Tipo             | Descripción                          |
|--------------------|------------------|--------------------------------------|
| `df`               | pd.DataFrame     | Salida de `leer_entrada`             |
| `mapping_productos`| Dict[str, int]   | `{nombre_producto: código}`          |
| `mapping_fuentes`  | Dict[str, int]   | `{nombre_mercado: código}`           |
| `mapping_grupos`   | Dict[str, int]   | `{nombre_grupo: código}`             |

**Returns:** DataFrame con columnas originales más `Rproducto`, `RFuente`, `RGrupo`.

**Advertencias en consola:**
- `WARNING: X filas excluidas por productos sin código` — productos nuevos no en `productos.yml`.
- `WARNING: X filas con fuentes sin código` — mercados nuevos no en `fuentes.yml`.

---

## Pipeline `aggregation` — `src/sipsa/pipelines/aggregation/nodes.py`

### `armar_cuadros(df, mapping_productos, config_cuadros, orden_mercados) → pd.DataFrame`

Construye el DataFrame completo del boletín en el orden exacto del SAS.
Para cada cuadro: título → cabecera de columnas → productos (nombre + mercados + separador).

**Args:**

| Parámetro          | Tipo                    | Descripción                                    |
|--------------------|-------------------------|------------------------------------------------|
| `df`               | pd.DataFrame            | Salida de `aplicar_mappings`                  |
| `mapping_productos`| Dict[str, int]          | Para obtener nombre a partir del código        |
| `config_cuadros`   | Dict[str, Any]          | Contenido de `cuadros.yml`                     |
| `orden_mercados`   | OrdenMercadosStrategy   | Estrategia de ordenamiento (opcional)          |

**Returns:** DataFrame con columna `_tipo` que indica cómo renderizar cada fila.

**Patrón Strategy — `OrdenMercadosStrategy`:**

| Clase                           | Comportamiento                                                  |
|---------------------------------|-----------------------------------------------------------------|
| `OrdenPorCodigoLuegoAlfabetico` | Mercados con código: ascendente por RFuente. Sin código: alfabético al final. (Por defecto) |

Para implementar un nuevo criterio de ordenamiento:
```python
class OrdenPorPrecioDescendente(OrdenMercadosStrategy):
    def ordenar(self, datos_producto: pd.DataFrame) -> pd.DataFrame:
        return datos_producto.sort_values("P(1)", ascending=False)

# Pasarlo al nodo:
armar_cuadros(df, mapping_productos, config_cuadros, OrdenPorPrecioDescendente())
```

---

## Pipeline `reporting` — `src/sipsa/pipelines/reporting/nodes.py`

### `generar_excel(boletin_filas, fecha, ruta_salida) → Dict[str, Any]`

Escribe el boletín formateado en Excel usando `openpyxl`. Reemplaza completamente
la macro VBA original.

**Args:**

| Parámetro      | Tipo         | Descripción                                     |
|----------------|--------------|-------------------------------------------------|
| `boletin_filas`| pd.DataFrame | Salida de `armar_cuadros`                      |
| `fecha`        | str          | Ej. `"27FEB2026"` → nombre del archivo de salida |
| `ruta_salida`  | str          | Ej. `"data/08_reporting"`                       |

**Returns:** Dict con metadata del archivo generado (ruta, filas escritas).

**Patrón Strategy — `FilaStrategy`:**

| Clase              | Tipo de fila  | Estilo aplicado                                |
|--------------------|---------------|------------------------------------------------|
| `CuadroStrategy`   | `cuadro`      | Negrita, texto del título del cuadro           |
| `ColumnasStrategy` | `columnas`    | Negrita, 5 encabezados de columna              |
| `ProductoStrategy` | `producto`    | Negrita, nombre del producto                   |
| `DatoStrategy`     | `dato`        | Normal, precios con separador de miles         |
| `SeparadorStrategy`| `separador`   | Fila en blanco (6 pt de alto)                  |

Para añadir un nuevo tipo de fila:
```python
class MiNuevoTipo(FilaStrategy):
    def renderizar(self, ws, fila, row):
        ws.cell(row=fila, column=1, value="texto")

# Registrarlo:
_ESTRATEGIAS["mi_tipo"] = MiNuevoTipo()
```

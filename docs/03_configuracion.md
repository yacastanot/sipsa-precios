# Variables de Configuración

## Archivo `.env`

El archivo `.env` en la raíz del proyecto concentra **todos los valores que cambian
entre ejecuciones**. No se sube al repositorio (está en `.gitignore`).
Copiar `.env.example` y completar los valores antes de cada ejecución.

### Variables del pipeline semanal

| Variable          | Tipo   | Ejemplo              | Propósito                                                   |
|-------------------|--------|----------------------|-------------------------------------------------------------|
| `SIPSA_FECHA`     | string | `06MAR2026`          | Fecha del boletín. Aparece en el nombre del archivo Excel de salida. Formato: `DDMMMYYYY` en mayúsculas. |
| `SIPSA_ARCHIVO`   | string | `Listado a 06 mar 26.xlsx` | Nombre exacto del archivo Excel en `data/01_raw/`. Debe coincidir byte a byte (mayúsculas, espacios, extensión). |

## Archivo `conf/base/parameters.yml`

Lee las variables de entorno mediante el resolver `env:` de OmegaConf,
registrado en `src/sipsa/settings.py`.

```yaml
fecha:           ${env:SIPSA_FECHA}
archivo_entrada: ${env:SIPSA_ARCHIVO}
ruta_entrada:    "data/01_raw"
ruta_salida:     "data/08_reporting"
```

| Parámetro        | Quién lo usa          | Propósito                           |
|------------------|-----------------------|-------------------------------------|
| `fecha`          | nodo `generar_excel`  | Nombre del archivo de salida        |
| `archivo_entrada`| nodo `leer_entrada`   | Nombre del Excel que se leerá       |
| `ruta_entrada`   | nodo `leer_entrada`   | Directorio del Excel de entrada     |
| `ruta_salida`    | nodo `generar_excel`  | Directorio donde se guarda el Excel |

## Catálogos de mappings (`conf/base/mappings/`)

Estos YAML son los catálogos maestros. Se actualizan sólo cuando cambia el universo
de productos o mercados, no cada semana.

### `productos.yml`

Mapea el nombre exacto del producto al código numérico de 8 dígitos (`Rproducto`).

**Estructura del código:** `[Grupo][Subgrupo][Tipo][Secuencia]`
- `10121001` → Acelga (Cuadro 1, hortalizas hoja verde)
- `60131001` → Alas de pollo (Cuadro 6, pollo)

> Los nombres deben coincidir byte a byte con el Excel. El Excel usa codepoints
> Latin-1 almacenados como Unicode: `á=0xe1`, `é=0xe9`, `í=0xed`, `ñ=0xf1`, `ó=0xf3`, `ú=0xfa`.

Para agregar un producto nuevo:
1. Añadir en `productos.yml` con código apropiado.
2. Añadir el código al cuadro correspondiente en `cuadros.yml`.

### `fuentes.yml`

Mapea el nombre del mercado al código numérico (`RFuente`, rango 1001–1134).
Los mercados sin código se incluyen al final de cada producto, sin orden específico.

Para agregar un mercado nuevo: añadir en `fuentes.yml` con el siguiente código disponible.

### `grupos.yml`

Mapea los 8 grupos de alimentos a códigos 1–8. Rara vez cambia.

| Código | Grupo                                   |
|--------|-----------------------------------------|
| 1      | Verduras y hortalizas                   |
| 2      | Frutas frescas                          |
| 3      | Tubérculos y plátanos                   |
| 4      | Granos y cereales                       |
| 5      | Huevos, lácteos y derivados             |
| 6      | Carnes y pescados                       |
| 7      | Panela, azúcar y otros                  |
| 8      | Aceites y grasas                        |

### `cuadros.yml`

Define el orden exacto de productos dentro de cada cuadro del boletín.
Cada entrada es una lista de `Rproducto` en el orden en que deben aparecer.
Los productos sin datos en la semana se omiten automáticamente.

## Archivo `conf/base/catalog.yml`

Registra todos los datasets del pipeline. Los más relevantes:

| Dataset               | Tipo                     | Ruta / Origen                    |
|-----------------------|--------------------------|----------------------------------|
| `precios_entrada`     | `MemoryDataset`          | No persiste entre runs           |
| `precios_transformados` | `pandas.CSVDataset`    | `data/02_intermediate/`          |
| `boletin_filas`       | `pandas.CSVDataset`      | `data/03_primary/`               |
| `mapping_productos`   | `yaml.YAMLDataset`       | `conf/base/mappings/productos.yml` |
| `mapping_fuentes`     | `yaml.YAMLDataset`       | `conf/base/mappings/fuentes.yml` |
| `mapping_grupos`      | `yaml.YAMLDataset`       | `conf/base/mappings/grupos.yml`  |
| `mapping_cuadros`     | `yaml.YAMLDataset`       | `conf/base/mappings/cuadros.yml` |

> Los mappings se cargan como `yaml.YAMLDataset` (no como `params:`) para evitar
> archivos de parámetros de cientos de líneas. Ver [01_arquitectura.md](01_arquitectura.md).

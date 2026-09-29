# Arquitectura y Decisiones de Diseño

## ¿Por qué Kedro?

El proceso original combinaba SAS (lógica de datos) y una macro VBA (formato Excel), lo que
generaba dependencias de licencias costosas y dificultaba la reproducibilidad. Kedro fue elegido
porque:

- **Reproducibilidad**: cada run es stateless y determinista.
- **Modularidad**: separa lectura, transformación, agregación y reporte en pipelines independientes.
- **Trazabilidad**: el catálogo registra todas las entradas y salidas con sus rutas exactas.
- **Parametrización**: `parameters.yml` + `.env` permiten cambiar la semana sin tocar el código.

## Equivalencia SAS → Python

| Componente original              | Equivalente Python                          |
|----------------------------------|---------------------------------------------|
| `SIPSA_PRECIO_SEMANAL.sas`       | 4 pipelines en `src/sipsa/pipelines/`       |
| `Macro_Excel_Formato_Inf_Semanal_SIPSA_P.txt` | Nodo `generar_excel` con openpyxl |
| Variables SAS `Rproducto`, `RFuente`, `RGrupo` | Columnas del DataFrame tras `aplicar_mappings` |
| `DATA Boletin1` (codificación)   | Pipeline `transformation`                   |
| `%Prod`, `DATA Cuadro1-8`        | Pipeline `aggregation` + `cuadros.yml`      |

## Arquitectura en capas (Medallion)

```
data/01_raw/          ← Bronze  : Excel semanal crudo, sin transformar
data/02_intermediate/ ← Silver  : precios con códigos numéricos (Rproducto, RFuente, RGrupo)
data/03_primary/      ← Silver+ : filas del boletín ordenadas y tipadas
data/08_reporting/    ← Gold    : Excel formateado listo para publicar
```

## Diagrama de componentes

```
.env
 │  SIPSA_FECHA, SIPSA_ARCHIVO
 ▼
conf/base/parameters.yml  ──────────────────────────────────────┐
                                                                 │
conf/base/mappings/                                              │
  productos.yml ──┐                                             │
  fuentes.yml  ──┤  catalog.yml                                 │
  grupos.yml   ──┤  (yaml.YAMLDataset)                          │
  cuadros.yml  ──┘       │                                      │
                          │                                      │
                     ┌────▼────────────────────────────────────┐│
                     │  Pipeline: ingestion                     ││
                     │  leer_entrada() → precios_entrada        ││
                     └────┬────────────────────────────────────┘│
                          │                                      │
                     ┌────▼────────────────────────────────────┐│
                     │  Pipeline: transformation                 ││
                     │  aplicar_mappings() → precios_transform. ││
                     └────┬────────────────────────────────────┘│
                          │                                      │
                     ┌────▼────────────────────────────────────┐│
                     │  Pipeline: aggregation                    ││
                     │  armar_cuadros() → boletin_filas         ││
                     └────┬────────────────────────────────────┘│
                          │                                      │
                     ┌────▼────────────────────────────────────┐│
                     │  Pipeline: reporting                      ││
                     │  generar_excel() → Boletin_*.xlsx  ◄─────┘│
                     └─────────────────────────────────────────┘
```

## Patrón Strategy (por qué se usó)

Dos partes del pipeline tienen variantes intercambiables sin cambiar el nodo principal:

**`aggregation` — OrdenMercadosStrategy**
El criterio de ordenamiento de mercados dentro de cada producto puede cambiar
(por código, por precio, alfabético). Implementar una nueva subclase de
`OrdenMercadosStrategy` y pasarla a `armar_cuadros` es suficiente — sin tocar
la lógica del nodo.

**`reporting` — FilaStrategy**
Cada tipo de fila (`cuadro`, `columnas`, `producto`, `dato`, `separador`) tiene
su propia clase de renderizado. Añadir un nuevo tipo de fila sólo requiere crear
una subclase de `FilaStrategy` y registrarla en `_ESTRATEGIAS`.

## Stack tecnológico

| Librería          | Versión  | Rol                                      |
|-------------------|----------|------------------------------------------|
| Python            | 3.9+     | Lenguaje base                            |
| kedro             | 0.19.x   | Orquestación de pipelines                |
| kedro-datasets    | 7.x      | Conectores: Excel, Parquet, YAML         |
| pandas            | 2.x      | Manipulación de DataFrames               |
| openpyxl          | 3.1.x    | Generación del Excel formateado          |
| python-dotenv     | 1.x      | Carga de variables de entorno desde .env |
| fastapi           | 0.100+   | API web opcional                         |
| uvicorn           | 0.20+    | Servidor ASGI para la API                |

# Instalación y Configuración del Ambiente

## Requisitos del sistema

| Componente  | Versión mínima | Notas                              |
|-------------|----------------|------------------------------------|
| Python      | 3.9            | Probado con 3.13.9                 |
| pip         | 23+            | Incluido con Python                |
| S.O.        | Windows 10+    | También compatible con Linux       |
| RAM         | 4 GB           | El pipeline usa ~200 MB en ejecución |

No se requiere SAS, ni macro VBA, ni ninguna licencia de software privativo.

## Pasos de instalación

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd sipsa
```

### 2. Crear el entorno virtual

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate
```

### 3. Instalar dependencias

```bash
pip install -e .
```

Esto instala el proyecto en modo editable junto con todas las dependencias declaradas
en `pyproject.toml`.

### 4. Configurar variables de entorno

```bash
cp .env.example .env
```

Editar `.env` con los valores de la semana actual. Ver
[03_configuracion.md](03_configuracion.md) para la descripción de cada variable.

### 5. Verificar la instalación

```bash
kedro info
```

Debe mostrar la versión de Kedro y el nombre del proyecto (`sipsa`) sin errores.

## Estructura de carpetas de datos

Las carpetas de datos se crean automáticamente en el primer `kedro run`. Si necesita
crearlas manualmente:

```
data/
├── 01_raw/          # Pegar aquí el Excel semanal
├── 02_intermediate/ # Generado automáticamente
├── 03_primary/      # Generado automáticamente
└── 08_reporting/    # Aquí aparece el boletín final
```

## Uso de la API web (opcional)

Si se va a usar la interfaz web en lugar de la terminal:

```bash
# Desarrollo
uvicorn app:app --reload

# Producción (exponer en red)
uvicorn app:app --host 0.0.0.0 --port 8000
```

Requiere además las variables `SIPSA_USER` y `SIPSA_PASS` en el `.env`.
Ver [03_configuracion.md](03_configuracion.md).

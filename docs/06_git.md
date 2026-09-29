# Convenciones Git

## Estrategia de ramas

El repositorio usa una estrategia simple de dos ramas principales:

| Rama      | Propósito                                              |
|-----------|--------------------------------------------------------|
| `main`    | Código estable, listo para producción                  |
| `develop` | Integración de cambios en desarrollo                   |

Las ramas de trabajo se crean desde `develop` y se fusionan de vuelta a `develop`
mediante Pull Request. Sólo `develop` fusiona a `main` cuando el cambio está validado.

## Nomenclatura de ramas

```
<tipo>/<descripcion-breve>
```

| Tipo       | Cuándo usarlo                                   | Ejemplo                             |
|------------|-------------------------------------------------|-------------------------------------|
| `feature/` | Nueva funcionalidad o mejora                    | `feature/agregar-cuadro-9`          |
| `fix/`     | Corrección de bug                               | `fix/acento-ahuyamin`               |
| `chore/`   | Cambios de configuración, dependencias, CI      | `chore/actualizar-requirements`     |
| `docs/`    | Solo documentación                              | `docs/agregar-flujo-datos`          |
| `refactor/`| Reestructuración sin cambio de comportamiento   | `refactor/strategy-pattern-excel`   |

## Mensajes de commit

Formato:
```
<tipo>: <descripcion en presente, minúsculas, sin punto final>
```

Tipos válidos: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `style`.

**Ejemplos:**
```
feat: agregar soporte para Cuadro 9 en cuadros.yml
fix: corregir acento en producto Ahuyamín
docs: documentar patrón Strategy en 04_modulos.md
chore: actualizar openpyxl a 3.1.5
refactor: extraer helper _fila en aggregation/nodes.py
```

El mensaje dice **qué** cambió y **por qué** (si no es obvio). No describe **cómo**
(eso está en el código).

## Checklist de un Pull Request completo

Antes de abrir un PR, verificar:

- [ ] Código nuevo o modificado en `src/` o `conf/`
- [ ] Docstrings actualizados o agregados en las funciones modificadas
- [ ] `docs/04_modulos.md` actualizado si se añadió o cambió un módulo
- [ ] `docs/03_configuracion.md` actualizado si hay nuevas variables
- [ ] `.env.example` actualizado con las nuevas claves (sin valores reales)
- [ ] `requirements.txt` / `pyproject.toml` actualizados si hay nueva dependencia

Un PR sin documentación no está completo.

## Proceso de revisión y merge

1. Crear rama desde `develop`.
2. Hacer commits atómicos (un cambio por commit).
3. Abrir PR hacia `develop` con título descriptivo.
4. Al menos un revisor debe aprobar antes de fusionar.
5. Usar **Squash and merge** para mantener el historial limpio en `develop`.
6. El merge a `main` requiere que `develop` esté en estado verde (tests pasando).

## Archivos que NO se suben al repositorio

Definidos en `.gitignore`:

| Archivo / Carpeta     | Razón                                              |
|-----------------------|----------------------------------------------------|
| `.env`                | Contiene credenciales y parámetros semanales       |
| `data/01_raw/`        | Datos de entrada (pueden tener información sensible)|
| `data/02_intermediate/` | Generados automáticamente, no son fuente de verdad |
| `data/03_primary/`    | Idem                                               |
| `data/08_reporting/`  | Boletines finales (se distribuyen por otro canal)  |
| `.venv/`              | Entorno virtual local                              |

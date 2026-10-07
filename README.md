# SIPSA Precios — Pipeline Semanal

Automatización del boletín semanal de precios mayoristas SIPSA (DANE).
Migra el proceso original SAS + macro VBA a un pipeline reproducible en Python con Kedro.

**Estado:** Estable

## Inicio rápido

```bash
# 1. Copiar el Excel semanal a data/01_raw/
# 2. Configurar el ambiente
cp .env.example .env          # completar SIPSA_FECHA y SIPSA_ARCHIVO
pip install -e .
# 3. Ejecutar
kedro run
# Resultado en: data/08_reporting/Boletin_{fecha}_python.xlsx
```

## Requisitos

| Componente      | Versión mínima |
|-----------------|----------------|
| Python          | 3.9+           |
| kedro           | 0.19.x         |
| pandas          | 2.0+           |
| openpyxl        | 3.1+           |
| S.O.            | Windows / Linux |

## Documentación completa

- Manual de Usuario (PPTX) — disponible localmente en `docs/`, no versionado en el repositorio
- [Arquitectura y decisiones de diseño](docs/01_arquitectura.md)
- [Instalación y configuración del ambiente](docs/02_instalacion.md)
- [Variables de configuración (.env)](docs/03_configuracion.md)
- [Módulos y funciones](docs/04_modulos.md)
- [Flujo de datos extremo a extremo](docs/05_flujo_datos.md)
- [Convenciones Git](docs/06_git.md)

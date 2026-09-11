# ABAQ — Documentación

Sitio de documentación técnica y operativa de la plataforma **ABAQ**, construido con [Zensical](https://zensical.org).

## Estructura

```
docs/
├── docs/                          # Contenido Markdown del sitio
│   ├── index.md                   # Página de inicio
│   ├── architecture.md            # Arquitectura del sistema
│   ├── srs.md                     # Especificación de requisitos (SRS)
│   ├── tasks.md                   # Tareas y progreso
│   └── codebase-reading-guide.md  # Guía de lectura del código
├── reports/                       # Reportes de avance por entrega
│   └── report2-2026-09-09/
│       ├── attachtments/          # Capturas de pantalla
│       └── latex/                 # Fuente LaTeX, PDF y DOCX
├── zensical.toml                  # Configuración del sitio
├── pyproject.toml                 # Dependencias Python (uv)
└── .github/workflows/docs.yml     # CI/CD — deploy a GitHub Pages
```

## Requisitos

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)

## Instalación

```bash
uv sync
```

## Desarrollo local

```bash
uv run zensical serve
```

Disponible en `http://localhost:8000` con recarga en vivo.

## Build

```bash
uv run zensical build --clean
```

El sitio generado se guarda en `site/` (excluido del repositorio).

## Deploy

El deploy a GitHub Pages ocurre automáticamente al hacer push a `main` o `master` via `.github/workflows/docs.yml`.

## Versión actual

**v1.1.0** — Septiembre 2026

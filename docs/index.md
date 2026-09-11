---
icon: lucide/book-open
---

# ABAQ — Documentación

Bienvenido al sitio de documentación de **ABAQ**, la plataforma digital para la gestión del servicio social universitario.

## ¿Qué es ABAQ?

ABAQ es una plataforma web que centraliza la administración del servicio social para estudiantes universitarios y sus coordinadores. Permite registrar estudiantes, gestionar actividades, validar horas y llevar el seguimiento documental de cada expediente.

## Secciones de esta documentación

| Sección | Descripción |
|---|---|
| [Arquitectura](architecture.md) | Diseño técnico del sistema, componentes y decisiones de arquitectura |
| [Requisitos del sistema (SRS)](srs.md) | Especificación de requerimientos funcionales y no funcionales |
| [Tareas y progreso](tasks.md) | Listado de tareas de desarrollo y estado actual del proyecto |
| [Guía de lectura del código](codebase-reading-guide.md) | Mapa del código fuente para nuevos colaboradores |

## Estado del proyecto

!!! info "Versión actual: v1.1.0"
    El proyecto se encuentra en desarrollo activo en la rama `dev`.
    Los cambios más recientes incluyen protección de rutas con JWT en el backend
    y nuevas pantallas de administración en el frontend.

## Tecnologías

=== "Backend"
    - **Node.js** con Express
    - **MongoDB** (Atlas) con Mongoose
    - **JWT** para autenticación
    - **GridFS** para almacenamiento de archivos

=== "Frontend"
    - **Angular** (standalone components)
    - **SCSS** para estilos
    - **RxJS** para programación reactiva

=== "Infraestructura"
    - **GitHub Actions** para CI/CD
    - **Zensical** para esta documentación

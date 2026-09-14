::: titlepage
**ABAQ**\
Plataforma de Servicio Social

------------------------------------------------------------------------

**Reporte de Avance No. 1**\
Presentación inicial del prototipo y definición de requerimientos

------------------------------------------------------------------------

  -------------------- ----------------------------
            **Fecha:** 19 de agosto de 2026
          **Versión:** v1.0.0
           **Estado:** En planeación / Prototipado
    **Preparado por:** Equipo de Desarrollo ABAQ
  -------------------- ----------------------------

Documento confidencial --- Solo para uso interno del cliente
:::

# Resumen ejecutivo {#resumen-ejecutivo .unnumbered}

::: onehalfspace
Este documento presenta el primer reporte de avance del proyecto ABAQ. Durante este período se llevó a cabo la presentación oficial del primer prototipo funcional de la plataforma a la asociación ABAQ Querétaro, logrando alinear las expectativas sobre el diseño y la arquitectura del sistema. Se acordó el desarrollo de un sistema con accesos separados para administradores y estudiantes, garantizando la seguridad de la información. Finalmente, se negoció un esquema de trabajo de 8 entregas quincenales para la liberación del servicio social.
:::

# Introducción

Este documento resume los acuerdos y requerimientos recabados en la primera reunión oficial de seguimiento (19 de agosto de 2026) entre el equipo de desarrollo de software y ABAQ Querétaro. El propósito principal de la sesión fue validar el rumbo técnico del proyecto y formalizar las condiciones operativas.

# Cambios en el Servidor (Backend)

## Resumen general

En esta etapa, se definió la arquitectura base para la seguridad y separación de perfiles en la plataforma.

## Protección de rutas con autenticación

Se acordó que el sistema requerirá autenticación obligatoria. El acceso de los estudiantes a módulos administrativos estará bloqueado a nivel de sistema.

## Mejoras en la configuración

*Pendiente por implementar / Falta de información para esta etapa.*

  **Área**               Descripción del cambio
  ---------------------- ------------------------------------------------------------------------------------------------------
  **Seguridad**          Definición de separación estricta entre perfiles de Administrador y Estudiante.
  **Infraestructura**    Validación del almacenamiento en MongoDB (500 MB) como escalable y suficiente.

  : Resumen de cambios en el servidor (Backend) {#tab:backend-changes}

# Cambios en la Aplicación Web (Frontend)

## Resumen general

Se presentó el prototipo visual y se recogieron los requerimientos para los siguientes módulos del sistema.

## Panel de Administración

*Pendiente. Se presentará la distinción de vistas en la próxima sesión.*

## Vista de Detalle de Estudiante

*Falta de información / Pendiente de implementación.*

## Vista de Lista de Estudiantes

*Pendiente. Se acordó alimentar la plataforma con datos de prueba realistas para la próxima sesión.*

## Lista de Actividades

Se establecieron los requerimientos para la gestión de campañas (ej. jornadas de esterilización) y el registro de asistencia con control de entradas y salidas.

## Panel del Estudiante (Vista de inicio)

*Pendiente por presentar en la siguiente revisión.*

## Carga de Archivos

Se solicitó la capacidad de automatizar la generación de constancias de término o cartas de participación desde el sistema de forma digital.

  **Área**                      Descripción del cambio
  ----------------------------- --------------------------------------------------------------
  **Prototipo**                 Presentación y validación del diseño visual inicial con ABAQ Querétaro.
  **Requerimientos**            Definición de funcionalidades para campañas y generación de constancias.

  : Resumen de cambios en la aplicación web (Frontend) {#tab:frontend-changes}

# Evidencia Visual --- Capturas de Pantalla

*Falta de información. Pendiente de anexar capturas oficiales de las pantallas mostradas en esta sesión.*

# Estado Actual del Proyecto

::: center
*En Prototipado y Negociación. Se validó el prototipo funcional inicial y se acordó formalmente la liberación del servicio social a cambio de la entrega del sistema completo para diciembre de 2026.*
:::

# Próximos Pasos

1. **Vistas Separadas:** Desarrollar y presentar el panel dividido entre Administrador y Estudiante.
2. **Datos de Prueba:** Alimentar la plataforma con registros de prueba realistas.
3. **Documentación Formal:** Elaborar un documento formal que detalle la propuesta técnica y el calendario de entregas.

::: center

------------------------------------------------------------------------

\
ABAQ --- Reporte de Avance No. 1 · v1.0.0 · Agosto 2026
:::

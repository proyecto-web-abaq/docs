::: titlepage
**ABAQ**\
Plataforma de Servicio Social

------------------------------------------------------------------------

**Reporte de Avance No. 2**\
Cambios y mejoras recientes en la plataforma

------------------------------------------------------------------------

  -------------------- ----------------------------
            **Fecha:** 9 de septiembre de 2026
          **Versión:** v1.1.0
           **Estado:** En desarrollo (rama `dev`)
    **Preparado por:** Equipo de Desarrollo ABAQ
  -------------------- ----------------------------

Documento confidencial --- Solo para uso interno del cliente
:::

# Resumen ejecutivo {#resumen-ejecutivo .unnumbered}

::: onehalfspace
Este documento presenta el segundo reporte de avance del proyecto ABAQ, la plataforma digital para la gestión del servicio social. Durante este período se realizaron mejoras significativas tanto en la seguridad del servidor como en la experiencia visual de la aplicación web.

En el lado del servidor se implementó un sistema de autenticación que protege toda la información sensible: ahora solo los usuarios con sesión iniciada pueden consultar o modificar datos de estudiantes, contactos, actividades y encuestas. La única excepción intencional es el formulario público para aspirantes, que continúa disponible sin restricción.

En la aplicación web se construyeron tres pantallas completamente nuevas: el **Panel de Administración**, la **Vista de Detalle de Estudiante** y la **Vista de Detalle de Actividad**. Además se mejoraron las listas existentes, se rediseñó la página de inicio y se integró la capacidad de subir documentos directamente desde la plataforma.

El presente reporte documenta y evidencia estos avances con capturas de pantalla del sistema en funcionamiento, cumpliendo con el compromiso de coordinación y seguimiento del progreso establecido con el cliente.
:::

# Introducción

Este documento resume los cambios más recientes realizados en la plataforma **ABAQ**, la herramienta de gestión de servicio social. El objetivo es comunicar, de forma clara y sin tecnicismos, qué mejoró, qué se agregó y qué quedó más seguro desde la última entrega.

Los cambios se dividen en dos áreas principales:

- **Servidor (Backend):** la parte que no se ve, encargada de guardar y proteger los datos.

- **Aplicación web (Frontend):** la parte visual con la que interactúan administradores y estudiantes.

# Cambios en el Servidor (Backend)

## Resumen general

En esta etapa, el trabajo del servidor se enfocó en **seguridad**: asegurarse de que solo las personas que han iniciado sesión puedan ver o modificar la información de la plataforma.

## Protección de rutas con autenticación

Anteriormente, alguien con conocimientos técnicos podía acceder a la información de estudiantes, contactos, encuestas y actividades sin necesidad de haber iniciado sesión. Ahora eso ya no es posible.

Se protegieron los siguientes módulos:

- **Estudiantes:** para ver, crear, editar o eliminar un estudiante, es obligatorio haber iniciado sesión.

- **Contactos:** toda acción sobre contactos requiere autenticación.

- **Encuestas:** la consulta de encuestas ya guardadas ahora requiere sesión iniciada. La encuesta pública que llenan los aspirantes sigue disponible sin restricción.

- **Actividades y mascotas:** protegidas de la misma manera.

La única excepción intencional es el formulario de encuesta para aspirantes, que debe seguir siendo accesible para el público general.

## Mejoras en la configuración

- Se actualizó el archivo de ejemplo de configuración para incluir las variables necesarias para que el sistema de autenticación funcione correctamente.

- Se actualizó el archivo `README` con instrucciones más claras sobre cómo instalar y configurar el servidor.

- Se excluyó un archivo de registro local del control de versiones para evitar ruido innecesario.

  **Área**               Descripción del cambio
  ---------------------- ------------------------------------------------------------------------------------------------------
  **Seguridad**          Se requiere inicio de sesión para acceder a datos de estudiantes, contactos, encuestas y actividades
  **Encuesta pública**   El formulario para aspirantes continúa accesible sin sesión
  **Documentación**      README actualizado con instrucciones de instalación y variables de entorno
  **Configuración**      Archivo de ejemplo `.env.example` con variables de autenticación JWT
  **Limpieza**           Archivo `log.txt` excluido del repositorio

  : Resumen de cambios en el servidor (Backend) {#tab:backend-changes}

# Cambios en la Aplicación Web (Frontend)

## Resumen general

Esta es la parte con más cambios visibles. Se construyeron pantallas completamente nuevas y se mejoraron las ya existentes, con el objetivo de que tanto administradores como estudiantes tengan una experiencia más clara, completa y fácil de usar.

## Panel de Administración

Se creó una pantalla de inicio exclusiva para administradores. Desde ahí se puede ver de un vistazo cuántos estudiantes están registrados, cuántas actividades existen, cuántas inscripciones se han realizado y cuántas asignaciones están pendientes de validación. También incluye una sección de agenda con las próximas actividades programadas y un panel de seguimiento con elementos que requieren atención.

## Vista de Detalle de Estudiante

Se construyó una pantalla completa para consultar y editar la información de un estudiante en particular. Esta pantalla muestra el expediente completo: nombre, carrera, universidad, horas requeridas, horas acreditadas y horas pendientes. También permite editar la información, subir los tres documentos oficiales del servicio social (carta de aceptación, constancia de plática y carta de término), y consultar las actividades en las que el estudiante participa.

## Vista de Lista de Estudiantes

La pantalla de gestión de estudiantes fue mejorada. Ahora muestra una tabla con todos los alumnos registrados incluyendo nombre, carrera, correos, teléfono, horas por acreditar, horas validadas y periodo. Incluye un buscador por nombre, carrera, correo o teléfono, y acceso directo al detalle de cada estudiante.

## Lista de Actividades

Se mejoró la pantalla de actividades disponibles. Cada actividad muestra su tipo (plática, brigada de campo, etc.), duración en horas, descripción, ID, horario y enlace. Cuenta con buscador por título, descripción, tipo, horas o ID.

## Panel del Estudiante (Vista de inicio)

La pantalla de inicio para estudiantes fue rediseñada. Ahora muestra un saludo personalizado con el nombre del estudiante, su carrera, semestre, horas requeridas y horas acreditadas. También incluye secciones de actividades pendientes, historial de actividades y gráficas de progreso.

  **Área**                      Descripción del cambio
  ----------------------------- --------------------------------------------------------------
  **Panel de administración**   Nueva pantalla con métricas, agenda y panel de seguimiento
  **Detalle de estudiante**     Nueva pantalla con expediente, edición y carga de documentos
  **Lista de estudiantes**      Tabla mejorada con buscador y acceso a detalle
  **Lista de actividades**      Tarjetas mejoradas con buscador por múltiples criterios
  **Panel del estudiante**      Rediseño con datos personales, progreso e historial
  **Carga de archivos**         Nuevo servicio para subir documentos PDF desde la plataforma
  **Navegación**                Nuevas rutas para las pantallas de detalle

  : Resumen de cambios en la aplicación web (Frontend) {#tab:frontend-changes}

# Evidencia Visual --- Capturas de Pantalla

Las siguientes imágenes muestran el estado actual de la aplicación en la rama de desarrollo. Las capturas fueron tomadas el 10 de septiembre de 2026 y se presentan en el mismo orden que las secciones descritas en este reporte.

<figure id="fig:admin-home-top" data-latex-placement="H">
<img src="Capture-2026-09-10-180327.png" />
<figcaption>Panel de administración — métricas generales y próximas actividades (parte superior)</figcaption>
</figure>

<figure id="fig:admin-home-bottom" data-latex-placement="H">
<img src="Capture-2026-09-10-180430.png" />
<figcaption>Panel de administración — agenda, seguimiento y resumen de actividades (parte inferior)</figcaption>
</figure>

<figure id="fig:student-list" data-latex-placement="H">
<img src="Capture-2026-09-10-180521.png" />
<figcaption>Gestión de estudiantes — lista con buscador, horas acreditadas y acceso a detalle</figcaption>
</figure>

<figure id="fig:student-detail-top" data-latex-placement="H">
<img src="Capture-2026-09-10-180604.png" />
<figcaption>Expediente del estudiante — información completa y métricas de horas</figcaption>
</figure>

<figure id="fig:student-detail-docs" data-latex-placement="H">
<img src="Capture-2026-09-10-180639.png" />
<figcaption>Expediente del estudiante — sección de documentos y actividades</figcaption>
</figure>

<figure id="fig:student-detail-docs2" data-latex-placement="H">
<img src="Capture-2026-09-10-180830.png" />
<figcaption>Expediente del estudiante — documentos del expediente y actividades del estudiante</figcaption>
</figure>

<figure id="fig:activity-list-1" data-latex-placement="H">
<img src="Capture-2026-09-10-180920.png" />
<figcaption>Actividades disponibles — lista con buscador, tipo, duración y botón de registro</figcaption>
</figure>

<figure id="fig:activity-list-2" data-latex-placement="H">
<img src="Capture-2026-09-10-180950.png" />
<figcaption>Actividades disponibles — continuación de la lista</figcaption>
</figure>

<figure id="fig:student-home" data-latex-placement="H">
<img src="Capture-2026-09-10-181022.png" />
<figcaption>Panel del estudiante — bienvenida personalizada, carrera, horas y perfil</figcaption>
</figure>

<figure id="fig:student-pending" data-latex-placement="H">
<img src="Capture-2026-09-10-181109.png" />
<figcaption>Panel del estudiante — sección de actividades pendientes</figcaption>
</figure>

<figure id="fig:student-history" data-latex-placement="H">
<img src="Capture-2026-09-10-181123.png" />
<figcaption>Panel del estudiante — historial de actividades</figcaption>
</figure>

<figure id="fig:student-progress" data-latex-placement="H">
<img src="Capture-2026-09-10-181135.png" />
<figcaption>Panel del estudiante — progreso de servicio social y actividades asignadas</figcaption>
</figure>

# Estado Actual del Proyecto

::: center
*Sigue en discusión.*
:::

# Próximos Pasos

Los cambios descritos en este reporte han sido revisados, documentados y evidenciados con capturas de pantalla del sistema en funcionamiento. El compromiso de coordinación y seguimiento del progreso queda cubierto con la entrega de este documento.

No existen pendientes adicionales para esta etapa.

::: center

------------------------------------------------------------------------

\
ABAQ --- Reporte de Avance No. 2 · v1.1.0 · Septiembre 2026
:::

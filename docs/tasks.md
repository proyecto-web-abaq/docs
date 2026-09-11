# Plan de Ejecución y Matriz de Tareas (ISO/IEC/IEEE 12207:2017)
**Estándar:** ISO/IEC/IEEE 12207:2017 (Procesos del Ciclo de Vida del Software)  
**Proyecto:** Plataforma Web ABAQ (Servicio Social, Censo Comunitario y Documentación Pedagógica)  
**Módulos:** Arquitectura, Requerimientos, Comentarios JSDoc/TSDoc y Guía de Lectura  
**Versión:** 2.0.0  
**Fecha:** 2026-09-03  
**Estado:** Aprobado  
**Autores:** Equipo de Gestión y Calidad de Software ABAQ  

---

## 1. Visión General del Proceso de Ciclo de Vida

Conforme a la norma **ISO/IEC/IEEE 12207:2017**, este plan de ejecución estructura las tareas atómicas requeridas para el ciclo de vida de ingeniería, documentación de software y aseguramiento de la calidad (QA) del sistema ABAQ. El plan se organiza a través de los hitos **M0 a M4**, asegurando que ninguna modificación a código fuente se realice sin contar previamente con especificaciones arquitectónicas aprobadas, y que todas las adiciones documentales garanticen **cero regresión en el AST de ejecución (Zero Runtime Overhead)**.

---

## 2. Matriz de Tareas Atómicas de Ejecución (WBS)

| ID | Hito | Fase del Proceso | Descripción y Alcance de Archivos | Dependencias | Comando de Verificación | Estado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **TASK-DOC-01** | **M0** | Definición de Línea Base de Estándares | Armonizar y actualizar `docs/architecture.md` (IEEE 42010), `docs/srs.md` (IEEE 29148), `docs/tasks.md` (IEEE 12207) y `PROJECT.md` para reflejar la arquitectura integral y requerimientos de documentación. | Ninguna | `test -f docs/architecture.md && test -f docs/srs.md && test -f docs/tasks.md && test -f PROJECT.md` | ✅ Completado |
| **TASK-DOC-02** | **M1** | Autoría de la Guía Maestra de Lectura | Redactar `docs/codebase-reading-guide.md` en español conteniendo el Blueprint global, el ciclo de vida de Express, los patrones Mongoose ODM, la arquitectura Angular 22 Standalone y el flujo E2E. | TASK-DOC-01 | `test -f docs/codebase-reading-guide.md && grep -q "Mapa Mental" docs/codebase-reading-guide.md` | 📝 Listo para ejecutar |
| **TASK-DOC-03** | **M1** | Comentado JSDoc Backend: Núcleo y Seguridad | Documentar con JSDoc en inglés: `backend/app.js`, `models/user.model.js`, `controllers/auth.controller.js`, `routes/auth.route.js`, `middlewares/auth.middleware.js`, `utils/jwt.js`, `utils/appError.js`, `utils/catchAsync.js`. (8 archivos). | TASK-DOC-01 | `node --check backend/app.js backend/models/user.model.js backend/controllers/auth.controller.js backend/routes/auth.route.js backend/middlewares/auth.middleware.js backend/utils/jwt.js backend/utils/appError.js backend/utils/catchAsync.js` | 📝 Listo para ejecutar |
| **TASK-DOC-04** | **M2** | Comentado JSDoc Backend: Estudiantes y Encuestas | Documentar con JSDoc en inglés: `student.model.js`, `student.controller.js`, `student.route.js`, `student.validator.js`, `survey.model.js`, `survey.controller.js`, `survey.route.js`, `survey.validator.js`, `views/surveyForm.pug`. (9 archivos). | TASK-DOC-03 | `node --check backend/models/student.model.js backend/controllers/student.controller.js backend/routes/student.route.js backend/validators/student.validator.js backend/models/survey.model.js backend/controllers/survey.controller.js backend/routes/survey.route.js backend/validators/survey.validator.js` | 📝 Listo para ejecutar |
| **TASK-DOC-05** | **M2** | Comentado JSDoc Backend: Actividades, Mascotas y Utilidades | Documentar con JSDoc en inglés: `activity.model.js`, `activity.controller.js`, `activity.route.js`, `pet.model.js`, `pet.controller.js`, `pet.route.js`, `contact.model.js`, `contact.controller.js`, `contact.route.js`, `contact.validator.js`, `handler.controller.js`, `upload.controller.js`, `gridfs.js`, `email.js`, `validator.js`, `servicio.route.js`, `views/hello.pug`, `views/student.pug`. (19 archivos). | TASK-DOC-04 | `node --check backend/controllers/*.js backend/models/*.js backend/routes/*.js backend/middlewares/*.js backend/utils/*.js backend/validators/*.js` | 📝 Listo para ejecutar |
| **TASK-DOC-06** | **M3** | Comentado TSDoc Frontend: Núcleo y Configuración | Documentar con TSDoc en inglés: `main.ts`, `app.config.ts`, `app.routes.ts`, `app.component.ts`, `auth.guard.ts`, `auth.interceptor.ts`, `token.service.ts`, `auth.service.ts`, `fileupload.service.ts`, `modal.service.ts`, `modal.component.ts`, `not-found.component.ts`, `environments/*`. (14 archivos). | TASK-DOC-02 | `cd /Users/riosisraelg/Desktop/1/abaq/frontend && npx ng build --configuration development` | 📝 Listo para ejecutar |
| **TASK-DOC-07** | **M3** | Comentado TSDoc Frontend: Estudiantes y Encuestas | Documentar con TSDoc en inglés: `student.interfaces.ts`, `student.mapper.ts`, `student.service.ts`, `student.component.ts`, `student-detail.ts`, `home.component.ts`, `survey.interfaces.ts`, `survey.mapper.ts`, `survey.service.ts`, `survey.component.ts`, `form.component.ts`. (11 archivos). | TASK-DOC-06 | `cd /Users/riosisraelg/Desktop/1/abaq/frontend && npx ng build --configuration development` | 📝 Listo para ejecutar |
| **TASK-DOC-08** | **M3** | Comentado TSDoc Frontend: Actividades y Auxiliares | Documentar con TSDoc en inglés: `activity.interfaces.ts`, `activity.service.ts`, `activity.component.ts`, `activity-list.component.ts`, `activity-detail.ts`, `admin-home.ts`, `pet.interfaces.ts`, `pet.service.ts`, `pet-view.component.ts`, `contact.interfaces.ts`, `contact.service.ts`, `contact.component.ts`, `sobre-mi.ts`, `sobre-mi.component.ts`, `constants.ts`, `login.component.ts`, `signup.component.ts`, `admin-signup.component.ts`, `navbar.component.ts`. (19 archivos). | TASK-DOC-07 | `cd /Users/riosisraelg/Desktop/1/abaq/frontend && npx ng build --configuration development` | 📝 Listo para ejecutar |
| **TASK-DOC-09** | **M4** | Integración del Checklist Modular en la Guía | Consolidar la tabla interactiva de 80 archivos con casillas `- [ ]` y enlaces relativos dentro de `docs/codebase-reading-guide.md`, organizada en los 6 bloques de dominio. | TASK-DOC-05, TASK-DOC-08 | `python3 -c "import os, re; text=open('docs/codebase-reading-guide.md').read(); links=re.findall(r'\((../(?:backend|frontend)/[^\)]+)\)', text); missing=[l for l in links if not os.path.exists(os.path.join('docs', l))]; assert not missing, f'Missing: {missing}'; print('All links valid')"` | 📝 Listo para ejecutar |
| **TASK-DOC-10** | **M4** | Auditoría Final de Calidad y No Regresión (AST & Builds) | Ejecutar validación cruzada completa: sintaxis backend (`node --check`), compilación limpia frontend (`ng build`), verificación de rama git `docs-explanation` y revisión de integridad documental. | TASK-DOC-01 a TASK-DOC-09 | `node --check backend/app.js backend/controllers/*.js backend/models/*.js backend/routes/*.js backend/middlewares/*.js backend/utils/*.js backend/validators/*.js && cd /Users/riosisraelg/Desktop/1/abaq/frontend && npx ng build --configuration development && cd .. && git -C backend status -s && git -C frontend status -s` | 📝 Listo para ejecutar |

---

## 3. Protocolo de Transición y Criterios de Aceptación por Hito

### Hito M0: Línea Base de Ingeniería (Stage 1-3 IEEE)
- **Criterio de Salida**: `docs/architecture.md`, `docs/srs.md`, `docs/tasks.md` y `PROJECT.md` aprobados y alineados con las normas ISO/IEC/IEEE 42010:2022, 29148:2018 y 12207:2017.
- **Responsable**: Worker M0.

### Hito M1: Guía Maestra y Núcleo de Backend
- **Criterio de Salida**: `docs/codebase-reading-guide.md` creado con el Blueprint arquitectónico. 8 archivos base del backend documentados con JSDoc y verificados con `node --check`.

### Hito M2: Dominio Completo de Backend
- **Criterio de Salida**: 28 archivos restantes de backend documentados con JSDoc profesional. 100% de los 36 archivos backend superan `node --check`.

### Hito M3: Dominio Completo de Frontend
- **Criterio de Salida**: 44 archivos TypeScript documentados con TSDoc. El proyecto compila con `ng build` sin errores.

### Hito M4: Integración, Checklist y Auditoría Final
- **Criterio de Salida**: Checklist interactivo de 80 archivos operativo en la guía de lectura. Todos los enlaces relativos resuelven. Cero regresión en pruebas o compilación.

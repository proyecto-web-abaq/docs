# Documento de Arquitectura de Software (SAD)
**Estándar:** ISO/IEC/IEEE 42010:2022  
**Proyecto:** Plataforma Web ABAQ (Servicio Social, Censo Comunitario y Documentación Pedagógica)  
**Versión:** 2.0.0  
**Fecha:** 2026-09-03  
**Estado:** Aprobado  
**Autores:** Equipo de Arquitectura e Ingeniería de Software ABAQ  

---

## 1. Visión General del Sistema y Contexto (System Overview & Context)

### 1.1 Propósito y Misión del Sistema
La plataforma **ABAQ** es un sistema full-stack orientado a coordinar el servicio social universitario, el seguimiento y bienestar de animales comunitarios en colonias urbanas, y la gestión del expediente digital de estudiantes prestadores de servicio. El sistema permite:
1. El levantamiento censal y encuestas dinámicas de bienestar animal en campo (tanto por estudiantes como por ciudadanos).
2. La administración del expediente del estudiante (horas meta, horas validadas, estatus de documentos oficiales mediante almacenamiento binario en GridFS).
3. La gestión y asignación de convocatorias de actividades comunitarias, control de asistencia y acreditación precisa de horas de servicio.
4. El seguimiento y censo de mascotas registradas, campañas de esterilización, citas médicas veterinarias y red de contactos comunitarios.
5. Servir como plataforma de aprendizaje y referencia técnica ("código abierto pedagógico"), incorporando documentación técnica exhaustiva bajo estándares internacionales IEEE y comentarios en código rigurosos.

### 1.2 Partes Interesadas (Stakeholders) y Puntos de Vista
- **Estudiantes de Servicio Social**: Registran su perfil, seleccionan actividades comunitarias, suben comprobantes oficiales y consultan su progreso de horas en tiempo real con visualizaciones interactivas.
- **Administradores y Coordinadores Institucionales**: Crean convocatorias, validan asistencia y horas de servicio, auditan encuestas de bienestar animal, gestionan el censo veterinario y revisan expedientes documentales.
- **Comunidad / Encuestados**: Participan en censos comunitarios aportando información demográfica sobre mascotas y condiciones de hábitat animal a través de formularios reactivos web.
- **Desarrolladores e Investigadores Académicos**: Requieren una arquitectura desacoplada, autodocumentada y estrictamente estandarizada para mantenimiento continuo sin regresiones en tiempo de ejecución.

### 1.3 Pila Tecnológica Integral (Technology Stack)
- **Backend**:
  - Entorno de ejecución: Node.js v24 LTS.
  - Framework web: Express.js v4.19 con arquitectura modular (Routers, Middlewares, Controllers, Modelos, Validadores).
  - Capa ODM y Persistencia: Mongoose v8.4 sobre base de datos MongoDB (Atlas / local).
  - Almacenamiento binario: MongoDB GridFS (`GridFSBucket`) para segmentación y streaming de archivos grandes (PDFs, imágenes).
  - Motor de plantillas server-side: Pug (utilizado para vistas directas y formularios alternativos).
  - Criptografía y Seguridad: `bcryptjs` (hashing con factor de costo 12), `jsonwebtoken` (JWT stateless auth con HMAC-SHA256).
- **Frontend**:
  - Framework cliente: Angular v22 basado en componentes independientes (**Standalone Components**) sin `NgModule`.
  - Inyección de dependencias: Inyección funcional moderna con `inject()`.
  - Reactividad y gestión asíncrona: RxJS v7.8 (`BehaviorSubject`, operadores de transformación, limpieza con `takeUntilDestroyed`).
  - Formularios: Angular Reactive Forms (`FormBuilder`, `FormGroup`, `FormArray`, validadores personalizados).
  - Estilos y Maquetación: Bootstrap v5.3, Bootstrap Icons, SCSS modular con variables de marca ABAQ.
  - Visualización de datos: Chart.js v4.4 para gráficas reactivas de progreso y tendencias.

---

## 2. Vistas Arquitectónicas (ISO/IEC/IEEE 42010:2022)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       ARQUITECTURA DEL SISTEMA ABAQ                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [ CLIENTE WEB (Angular 22 Standalone) ]                                    │
│  ┌───────────────────────┐   ┌────────────────────────┐                     │
│  │ Vistas y Componentes  │   │ Servicios y Estado     │                     │
│  │ (Home, Student, Form) │<─>│ (Auth, Student, Survey)│                     │
│  └──────────┬────────────┘   └───────────┬────────────┘                     │
│             │                            │                                  │
│             ▼                            ▼                                  │
│  ┌───────────────────────┐   ┌────────────────────────┐                     │
│  │ Mappers DTO/Dominio   │   │ Interceptors & Guards  │                     │
│  │ (student, survey)     │   │ (auth.interceptor/guard)│                    │
│  └───────────────────────┘   └───────────┬────────────┘                     │
│                                          │ (HTTP / JSON / Bearer JWT)       │
│ ─────────────────────────────────────────┼───────────────────────────────── │
│                                          ▼                                  │
│  [ SERVIDOR BACKEND (Node.js 24 + Express 4.19) ]                           │
│  ┌────────────────────────────────────────────────────┐                     │
│  │ Express Pipeline: CORS -> Parsers -> Routers       │                     │
│  └───────────────────────┬────────────────────────────┘                     │
│                          ▼                                                  │
│  ┌───────────────────────┐   ┌────────────────────────┐                     │
│  │ Middlewares           │   │ Controladores y Factory│                     │
│  │ (auth, validator)     │──>│ (auth, student, handler)│                    │
│  └───────────────────────┘   └───────────┬────────────┘                     │
│                                          │                                  │
│                          ┌───────────────┴────────────────┐                 │
│                          ▼                                ▼                 │
│               ┌──────────────────────┐         ┌─────────────────────┐      │
│               │ Mongoose ODM Models  │         │ GridFS Bucket       │      │
│               │ (User, Student, etc.)│         │ (uploads.files/chnk)│      │
│               └──────────┬───────────┘         └──────────┬──────────┘      │
│ ─────────────────────────┼────────────────────────────────┼──────────────── │
│                          ▼                                ▼                 │
│  [ BASE DE DATOS (MongoDB Document & Binary Store) ]                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Vista Lógica (Logical View)
La vista lógica organiza el sistema en capas de responsabilidad única y desacoplada:

1. **Capa de Presentación (Frontend UI)**:
   - Reside en `frontend/src/app/components/`.
   - Implementada mediante **Angular Standalone Components**, eliminando la necesidad de módulos intermedios.
   - Cada componente encapsula su plantilla HTML, lógica TypeScript y estilos SCSS.
   - Componentes clave:
     - `LoginComponent` y `SignupComponent`: Autenticación y registro con secuenciación compatible con gestores de contraseñas.
     - `HomeComponent`: Dashboard reactivo con gráficos de avance de horas (`doughnut` y `line` charts).
     - `StudentComponent` y `StudentDetail`: Lista paginada/filtrable y expediente digital completo con visor de documentos PDF.
     - `FormComponent`: Formulario de encuestas dinámicas por lotes tipo tarjeta (*Typeform-style*).
     - `ActivityListComponent` y `ActivityDetail`: Convocatorias y pase de lista/acreditación de horas.

2. **Capa de Aplicación y Comunicación (Frontend Services & Mappers)**:
   - Servicios singleton (`@Injectable({ providedIn: 'root' })`) que centralizan la lógica de negocio del cliente y llamadas HTTP.
   - **Patrón Adaptador / DTO Mappers**:
     - `student.mapper.ts`: Traduce entre el contrato de base de datos en español (`BackendStudent`) y la interfaz tipada de Angular (`Student`).
     - `survey.mapper.ts`: Transforma estructuras dinámicas `FormGroup`/`FormArray` a un contrato estricto de 12 booleanos y conteos numéricos agregados (`BackendSurvey`).

3. **Capa de Transporte y Control (Backend API & Routers)**:
   - Reside en `backend/routes/` y `backend/controllers/`.
   - Enrutadores Express dedicados por dominio (`/auth`, `/student`, `/survey`, `/activity`, `/pet`, `/contact`, `/upload`).
   - Controladores que procesan la petición, delegan al modelo o servicio y generan respuestas HTTP normalizadas mediante `{ status: "success", data: ... }`.
   - **Patrón Controlador Factory (`handler.controller.js`)**: Generador dinámico de métodos CRUD estándar (`createOne`, `getOne`, `getAll`, `updateOne`, `deleteOne`), reduciendo duplicación de código.

4. **Capa de Dominio y Persistencia (Backend ODM & GridFS)**:
   - Modelos Mongoose (`backend/models/`):
     - `User`: Credenciales, rol (`student` | `admin`), métodos criptográficos estáticos y de instancia.
     - `Student`: Datos académicos, periodos y subdocumentos anidados `assignedActivities`.
     - `Survey`: Registro censal con normalización de booleanos mediante setters personalizados.
     - `Activity`, `Pet`, `Contact`: Catálogo de voluntariado, censo veterinario y bitácora de seguimiento.
   - **GridFS Bucket (`backend/utils/gridfs.js`)**: Almacenamiento y streaming de archivos binarios directamente sobre colecciones `uploads.files` y `uploads.chunks` de MongoDB.

### 2.2 Vista de Seguridad y Autenticación (Security / Auth View)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                VISTA DE SEGURIDAD Y CICLO DE AUTENTICACIÓN                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [ Flujo de Login ]                                                         │
│  Navegador ──POST /auth/login {correo, password}──> Express authRoute       │
│                                                           │                 │
│                                                           ▼                 │
│  bcrypt.compare(password, user.password) <────── authController.login       │
│            │ (Válido)                                                       │
│            ▼                                                                │
│  jwt.sign({ id: user._id }, SECRET, { expiresIn })                          │
│            │                                                                │
│            ▼                                                                │
│  Respuesta: JSON { status: "success", token, data: { user } }               │
│            │                                                                │
│            ▼ (Guardado en localStorage / memoria)                           │
│                                                                             │
│  [ Peticiones Subsecuentes Protegidas ]                                     │
│  HttpClient ──> auth.interceptor.ts (Inyecta Bearer <token>)                │
│                         │                                                   │
│                         ▼ (HTTP Header: Authorization: Bearer <token>)      │
│  Express auth.middleware.js (protect)                                       │
│         ├── 1. Extrae token del header Bearer                               │
│         ├── 2. jwt.verify(token, JWT_SECRET)                                │
│         ├── 3. User.findById(decoded.id)                                    │
│         └── 4. Asigna req.user = currentUser; next()                        │
│                                                                             │
│  [ Rollback Transaccional en Signup ]                                       │
│  authController.signup:                                                     │
│     Paso 1: User.create() exitoso                                           │
│     Paso 2: Student.create() lanza excepción                                │
│     Catch: User.findByIdAndDelete(user._id) ──> Previene cuentas huérfanas   │
└─────────────────────────────────────────────────────────────────────────────┘
```

- **Mecanismo de Autenticación**: Stateless basado en tokens JWT (JSON Web Tokens) firmados con algoritmo HMAC-SHA256 (`backend/utils/jwt.js`).
- **Protección de Credenciales**:
  - Hashing unidireccional con `bcryptjs`, factor de costo 12 (`salt rounds = 12`).
  - Campo de contraseña en esquema con `select: false` para evitar fugas accidentales en consultas.
  - Soporte de contraseñas de alta seguridad de hasta 128 caracteres, admitiendo frases de paso (*passphrases*) y caracteres especiales Unicode sin truncamiento.
- **Autorización y Control de Acceso Basado en Roles (RBAC)**:
  - Roles definidos: `student` y `admin`.
  - Registro de administradores (`POST /auth/admin/register`) protegido obligatoriamente por passkey secreta del entorno (`ADMIN_REGISTRATION_SECRET` / `ADMIN_PASSKEY`). Peticiones sin o con passkey inválida son rechazadas inmediatamente con código HTTP 403 Forbidden.
- **Defensas en Frontend**:
  - `auth.interceptor.ts`: Interceptor funcional que clona solicitudes HTTP para inyectar `Authorization: Bearer <token>` cuando el token existe, permitiendo peticiones públicas sin alteraciones.
  - `auth.guard.ts`:
    - `AuthGuard`: Restringe rutas privadas (`/home`, `/student`, `/activity`, `/survey`, `/admin/*`) redirigiendo a `/auth/login` si no hay sesión activa.
    - `PublicGuard`: Redirige a usuarios ya autenticados lejos de rutas de login/signup hacia su panel correspondiente (`/home` para alumnos, `/admin/home` para administradores).
  - Optimización para gestores de contraseñas (1Password, Bitwarden, iCloud Keychain, Chrome Autofill) mediante atributos `autocomplete="username"`, `autocomplete="current-password"` y `autocomplete="new-password"`.

### 2.3 Vista de Flujo de Datos y Ciclo de Vida (Data Flow & Lifecycle View)

#### A. Pipeline de Petición HTTP en Express (`backend/app.js`)
Cada petición recibida atraviesa una tubería secuencial estrictamente ordenada:

```
[Cliente HTTP / Navegador]
           │
           ▼
1. CORS Middleware (cors): Valida origen 'http://localhost:4200' y credentials: true
           │
           ▼
2. Body Parsers:
   - express.json(): Parsea cuerpos JSON (req.body)
   - express.urlencoded({ extended: true }): Parsea formularios HTML/Pug
           │
           ▼
3. Motor de Vistas (Pug):
   - app.set('view engine', 'pug'): Renderiza vistas servidor en /views
           │
           ▼
4. Enrutadores Montados (Routers):
   ├── /auth     ──> authRouter (login, signup, admin/register, me)
   ├── /student  ──> studentRouter (CRUD estudiantes, protegido por JWT)
   ├── /survey   ──> surveyRouter (envío público de encuestas + consulta admin)
   ├── /activity ──> activityRouter (CRUD actividades, asignación de horas)
   ├── /pet      ──> petRouter (censo de mascotas y citas)
   ├── /contact  ──> contactRouter (seguimiento y estatus de contacto)
   ├── /upload   ──> uploadRouter (subida y descarga de archivos con GridFS)
   └── /servicio ──> servicioRouter (stubs de servicio social)
           │
           ▼ (Si ninguna ruta coincide)
5. Capturador 404: app.all('*') ──> Genera new AppError(`${req.originalUrl} not found`, 404)
           │
           ▼ (En cualquier excepción en la cadena)
6. Middleware Global de Errores: (err, req, res, next) ──> JSON estandarizado { status, message }
```

#### B. Manejo de Errores Asíncronos (`catchAsync` y `AppError`)
- Para evitar bloques repetitivos `try/catch` en cada controlador, se implementa `catchAsync.js`:
  ```javascript
  module.exports = (fn) => (req, res, next) => {
    fn(req, res, next).catch(next);
  };
  ```
- Si una promesa es rechazada, `catch(next)` transfiere el error al middleware global de errores.
- Los errores previstos se instancian como `new AppError(mensaje, statusCode)` encapsulando el código HTTP y marcándose como operacionales (`isOperational = true`).

#### C. Ciclo de Streaming Binario con GridFS
1. **Subida**: El cliente envía `multipart/form-data`. `multer.memoryStorage()` retiene el búfer en memoria temporal. `upload.controller.js` abre un stream de escritura `bucket.openUploadStream(filename, { metadata })` y canaliza el búfer hacia MongoDB segmentándolo en fragmentos de 255KB.
2. **Descarga**: El cliente solicita `GET /upload/:fileId`. El controlador valida que el `fileId` sea un `ObjectId` válido de 24 caracteres hexadecimales. Si es válido, ejecuta `bucket.openDownloadStream(new ObjectId(fileId))` canalizándolo (`pipe(res)`) directamente a la respuesta HTTP.
3. **Limpieza de Huérfanos**: Si un archivo se sube a GridFS pero la posterior asociación con el expediente del estudiante falla en el cliente (`student-detail.ts`), el bloque `catch` invoca inmediatamente a `FileUploadService.deleteFile(url)` para evitar binarios huérfanos en la base de datos.

### 2.4 Vista de Topología de Componentes (Component Topology View)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     TOPOLOGÍA DE COMPONENTES FRONTEND                       │
├─────────────────────────────────────────────────────────────────────────────┤
│  Root: AppComponent (NavbarComponent + RouterOutlet)                        │
│                                                                             │
│  [ Rutas Públicas ]                                                         │
│  ├── LoginComponent ───────────────┐                                        │
│  ├── SignupComponent ──────────────┼──> AuthService ──> TokenService        │
│  └── FormComponent (Surveys) ──────┼──> SurveyService ──> survey.mapper     │
│                                    │                                        │
│  [ Rutas Privadas (AuthGuard) ]    │                                        │
│  ├── HomeComponent (Metrics/Charts)│                                        │
│  ├── StudentComponent ─────────────┼──> StudentService ──> student.mapper   │
│  ├── StudentDetail ────────────────┼──> FileUploadService (GridFS)          │
│  ├── ActivityListComponent ────────┼──> ActivityService                     │
│  ├── ActivityDetail ───────────────┘                                        │
│  ├── PetViewComponent ────────────────> PetService                          │
│  └── ContactComponent ────────────────> ContactService                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

- **Frontend (44 archivos fuente TypeScript)**:
  - 17 Componentes Standalone (`app`, `navbar`, `login`, `signup`, `admin-signup`, `home`, `admin-home`, `student`, `student-detail`, `activity`, `activity-list`, `activity-detail`, `form`, `survey`, `pet-view`, `contact`, `sobre-mi`, `modal`, `not-found`).
  - 9 Servicios (`AuthService`, `TokenService`, `StudentService`, `SurveyService`, `ActivityService`, `PetService`, `ContactService`, `FileUploadService`, `ModalService`).
  - 2 Mappers de Dominio/DTO (`student.mapper.ts`, `survey.mapper.ts`).
  - 2 Guards e Interceptores funcionales (`auth.guard.ts`, `auth.interceptor.ts`).
  - 7 Interfaces de modelo y constantes (`student`, `survey`, `activity`, `pet`, `contact`, `sobre-mi`, `constants`).
  - 3 Archivos de entorno (`environment.ts`, `environment.development.ts`, `environment.production.ts`).

- **Backend (36 archivos fuente JavaScript y Vistas)**:
  - 1 Servidor/Configuración central (`app.js`).
  - 8 Enrutadores (`auth`, `student`, `survey`, `activity`, `pet`, `contact`, `upload`, `servicio`).
  - 8 Controladores (`auth`, `student`, `survey`, `activity`, `pet`, `contact`, `upload`, `handler.controller`).
  - 6 Modelos Mongoose (`User`, `Student`, `Survey`, `Activity`, `Pet`, `Contact`).
  - 2 Middlewares (`auth.middleware.js`, `validator.js`).
  - 3 Validadores (`student.validator.js`, `survey.validator.js`, `contact.validator.js`).
  - 5 Utilidades (`jwt.js`, `appError.js`, `catchAsync.js`, `gridfs.js`, `email.js`).
  - 3 Vistas Pug (`hello.pug`, `student.pug`, `surveyForm.pug`).

### 2.5 Vista de Despliegue y Distribución (Deployment View)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VISTA DE DESPLIEGUE FÍSICO / DISTRIBUCIÓN                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   [ Dispositivo Cliente ]                                                   │
│   Navegador Web (Chrome, Firefox, Safari, Edge)                             │
│         │                                                                   │
│         │ HTTPS (Puerto 443 / 4200 en desarrollo)                           │
│         ▼                                                                   │
│   [ Servidor Web / Reverse Proxy / Nginx ]                                  │
│   ├── Archivos Estáticos Angular SPA (/dist/frontend/browser)                │
│   └── Proxy Inverso /api/ ──> Puerto 3000                                   │
│                                  │                                          │
│                                  ▼                                          │
│   [ Servidor de Aplicaciones (Node.js 24 Runtime) ]                         │
│   Proceso Express.js (backend/app.js)                                       │
│   ├── Variables de entorno (.env): PORT, MONGO_URI, JWT_SECRET, ADMIN_PASS  │
│   └── Conexión Mongoose Pool ─────────────────┐                             │
│                                               │                             │
│                                               ▼                             │
│   [ Clúster de Base de Datos (MongoDB Atlas / Local) ]                      │
│   ├── Colecciones Documentales: users, students, surveys, activities, etc.  │
│   └── Bucket GridFS: uploads.files, uploads.chunks (bloques de 255KB)       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

- **Aislamiento en Control de Versiones**: Todo el código de producción y documentación del proyecto está unificado y versionado en la rama `docs-explanation` tanto en el repositorio del backend como del frontend.

---

## 3. Registros de Decisiones Arquitectónicas (Architectural Decision Records - ADRs)

### ADR-01: Adopción de Componentes Independientes (Standalone) e Inyección Funcional en Angular 22
- **Estado**: Aceptado y Aplicado.
- **Contexto**: Las versiones previas de Angular requerían definir módulos (`NgModule`), introduciendo complejidad en la configuración, importaciones circulares y paquetes de distribución más pesados.
- **Decisión**: Se adopta el estándar de **Standalone Components** en el 100% de los componentes del frontend (`@Component({ standalone: true, imports: [...] })`), así como la función `inject()` para resolver dependencias (`AuthService`, `HttpClient`, `Router`) directamente en inicializadores de propiedades de clase.
- **Consecuencias**:
  - *Positivas*: Mayor modularidad, eliminación de archivos `*.module.ts`, tree-shaking optimizado por el compilador, menor fricción cognitiva para desarrolladores noveles.
  - *Negativas*: Requiere rigor en declarar explícitamente en el arreglo `imports` de cada componente las directivas o módulos de Angular (`CommonModule`, `ReactiveFormsModule`, `RouterModule`) que utiliza.

### ADR-02: Autenticación Desacoplada y Criptografía con JWT y RBAC
- **Estado**: Aceptado y Aplicado.
- **Contexto**: El sistema debe operar con un frontend SPA y una API RESTful que permitan escalabilidad horizontal sin persistencia de sesiones en memoria del servidor web (*stateless*).
- **Decisión**: Implementar autenticación basada en tokens JWT firmados con HMAC-SHA256 y expiración configurable. El control de acceso basado en roles (`student` vs `admin`) se valida tanto en el middleware del backend (`auth.middleware.js`) como en los guards del frontend (`auth.guard.ts`). El registro de administradores requiere forzosamente una clave secreta configurada en el entorno (`ADMIN_REGISTRATION_SECRET`).
- **Consecuencias**:
  - *Positivas*: Arquitectura completamente desacoplada; cero consumo de memoria de sesiones en el backend; soporte nativo para balanceo de carga; seguridad criptográfica estándar.
  - *Negativas*: La revocación inmediata de un token emitido antes de su expiración requiere listas de revocación en base de datos si se requiere invalidación forzada antes del vencimiento.

### ADR-03: Documentación Pedagógica en Código (JSDoc/TSDoc en Inglés) y Guía Arquitectónica en Español
- **Estado**: Aceptado y Aplicado.
- **Contexto**: El proyecto ABAQ es utilizado por estudiantes y docentes de servicio social técnico. Es prioritario que cualquier desarrollador pueda entender tanto la visión conceptual de alto nivel como los mecanismos internos de los frameworks (Express, Mongoose, Angular, RxJS).
- **Decisión**: Se establece una **disciplina bilingüe rigurosa**:
  1. La documentación conceptual, arquitectura (`docs/architecture.md`), requerimientos (`docs/srs.md`), tareas (`docs/tasks.md`) y guía de lectura interactiva (`docs/codebase-reading-guide.md`) se redactan en **español técnico**.
  2. Los comentarios en código fuente (JSDoc en backend, TSDoc en frontend) se redactan en **inglés técnico**, siguiendo los estándares internacionales de código abierto y herramientas de análisis estático.
  3. Los comentarios deben ser pedagógicos: explicar *por qué* existe el componente, su rol en el ciclo de vida y qué hace cada mecanismo de framework (`next()`, `catchAsync`, `takeUntilDestroyed`, `FormArray`, etc.), sin alterar el AST de ejecución (Zero Runtime Overhead).
- **Consecuencias**:
  - *Positivas*: Máxima claridad formativa y mantenibilidad a largo plazo; compatibilidad total con herramientas IDE (IntelliSense, TypeDoc, JSDoc); cero impacto en el rendimiento en tiempo de ejecución.
  - *Negativas*: Requiere esfuerzo continuo de autoría y auditoría para mantener sincronizados los comentarios con la evolución del código.

---

## 4. Atributos de Calidad del Sistema (Quality Attributes)

1. **Mantenibilidad y Comprensibilidad (Maintainability & Understandability)**:
   - Cobertura del 100% de los 80 archivos no plantilla con comentarios normalizados.
   - Existencia de un checklist interactivo por dominios que permite a nuevos ingenieros rastrear su avance formativo en el repositorio.
2. **Inocuidad Operativa (Zero Runtime Overhead / Zero AST Regression)**:
   - Los comentarios agregados no alteran identificadores, lógica de control, firmas de funciones ni estructuras de datos ejecutables.
   - Las pruebas y compilaciones del backend (`node --check`) y del frontend (`ng build`) se mantienen con código de salida 0.
3. **Seguridad y Confidencialidad (Security & Confidentiality)**:
   - Cifrado unidireccional bcrypt con factor de costo 12.
   - Contraseñas robustas de hasta 128 caracteres.
   - Prevención de fugas de contraseñas mediante `select: false` en consultas de Mongoose.
   - Passkey secreta en variables de entorno para alta administrativa.
4. **Usabilidad y Accesibilidad (Usability & Accessibility)**:
   - Formularios multietapa secuenciales estilo *Typeform* con soporte permanente de teclado (`Enter`, `Tab`, flechas).
   - Atributos estándar de autocompletado (`autocomplete`) para compatibilidad fluida con gestores de contraseñas.
5. **Confiabilidad y Manejo de Errores (Reliability & Fault Tolerance)**:
   - Manejo centralizado de excepciones operacionales (`AppError`).
   - Rollback transaccional ante fallos en creación de perfiles de usuario.
   - Limpieza automática de archivos huérfanos en GridFS ante fallos de persistencia en el cliente.

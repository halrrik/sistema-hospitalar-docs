# Prompt Maestro – Sistema Hospitalar v1.0

Documento maestro unificado del proyecto **Sistema Hospitalar** para una clínica en Perú. Compila y normaliza los resúmenes técnicos, elimina duplicidades y establece un estándar único de términos, versiones y decisiones. Idioma de documentación y comentarios de código: **español técnico (LatAm)**.

---

## Objetivo

Construir un sistema hospitalario modular con **backend y frontend separados** e integración vía **API REST**, priorizando simplicidad, seguridad, trazabilidad y despliegue asequible en **Hostinger**.

!!! note "Alcance del MVP"
- Gestión de **Usuarios/Roles/Permisos**
- **Pacientes**, **Médicos** y **Especialidades**
- **Convenios (Aseguradoras)** y **Planes**
- **Admisión** (apertura y seguimiento)
- **Citas** (agenda básica)
- Dashboard con KPIs mínimas y reportes simples (CSV/Excel)
- Auditoría esencial (timestamps, `user_id`)

---

## Contexto Operativo

* **Desarrollo local:** Windows + **XAMPP** (MySQL + phpMyAdmin) + **PowerShell**
* **Repositorio:** Git/GitHub
* **Producción:** **Hostinger** (PHP 8; docroot `public/`)
* **Frontend:** Laravel 12 + **AdminLTE** (Blade; **sin Vue**)
* **Backend API:** Laravel 12 (API v1 con Sanctum)

---

## Stack Tecnológico

| Capa          | Tecnología / Versión          | Notas clave                                                        |
| ------------- | ----------------------------- | ------------------------------------------------------------------ |
| Backend       | Laravel **12.x**, PHP **8.x** | Controllers API, Form Requests, Policies, Resources                |
| Frontend      | Laravel 12 + **AdminLTE**     | Blade, DataTables, jQuery selectivo, Bootstrap incluido            |
| Base de datos | **MySQL** (InnoDB, utf8mb4)   | Collation `utf8mb4_unicode_ci`, FK y `softDeletes`                 |
| Auth API      | **Laravel Sanctum**           | PATs con scopes por rol/permiso                                    |
| Herramientas  | Composer, NPM (assets), Git   | Windows PowerShell                                                 |
| Infra         | **Hostinger**                 | PHP 8, `storage:link`, `config:cache`, `route:cache`, `view:cache` |

---

## Arquitectura (Backend / Frontend / API)

### Diagrama (alto nivel)

```
[Frontend (Laravel + Blade + AdminLTE)]
        │  fetch/axios (Bearer <token>)
        ▼
     [API Gateway / Router Laravel]
        │  /api/v1/*
        ▼
 [Controladores API + Services + Policies]
        │
        ▼
    [Eloquent ORM / MySQL]
```

### Backend (API)

```
app/
  Http/
    Controllers/Api/V1/
    Requests/
    Middleware/
  Models/
  Policies/
  Http/Resources/
database/migrations/
database/seeders/
routes/api.php
```

* **Versión de API:** prefijo `/api/v1/...`
* **Validación:** `FormRequest` en `app/Http/Requests`
* **Autorización:** `Policies` + `Gates` por rol/permiso
* **Serialización:** `Resources` para respuestas consistentes
* **CORS:** múltiples orígenes (local y producción)
* **Rate limit:** por token (throttle)

### Frontend (Blade + AdminLTE)

* **Layout base:** `resources/views/layouts/adminlte.blade.php`
* **Parciales:** header, sidebar, breadcrumbs, toasts, modales
* **Vistas por módulo:** `resources/views/{modulo}/index|create|edit|show.blade.php`
* **Assets:** `public/assets` (CSS/JS), DataTables y Select2 cuando aplique
* **Consumo API:** `fetch`/`axios` con inyección de token Sanctum y manejo centralizado de errores

**Interceptor sugerido (axios):**

```js
// resources/js/api.js
import axios from 'axios';

const api = axios.create({ baseURL: '/api/v1', timeout: 15000 });

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('sh_token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (r) => r,
  (err) => {
    if (err?.response?.status === 401) window.location.href = '/login';
    return Promise.reject(err);
  }
);

export default api;
```

### Convenciones de Integración

* **RESTful:** paginación, filtros y sort en `GET`
* **Códigos HTTP:** 2xx éxito, 4xx errores del cliente, 5xx servidor
* **Formato JSON:**

```json
{
  "data": {},
  "meta": { "page": 1, "per_page": 15, "total": 120 },
  "errors": [
    { "code": "VALIDATION_ERROR", "title": "Campo inválido", "detail": "..." }
  ]
}
```

* **Fechas:** ISO 8601 (UTC) en API; UI en `America/Lima`
* **Idempotencia opcional:** `Idempotency-Key` en POST críticos

---

## Base de Datos

!!! info "Parámetros recomendados"
- Motor **InnoDB**
- Collation **utf8mb4_unicode_ci**
- `timestamps` y `softDeletes` en entidades clave

### Entidades núcleo (MVP)

* **users**: `id, name, email(unique), password(hash), status, last_login_at`
* **roles**, **permissions** y pivotes (o **Spatie Permissions**)
* **patients**: `id, document_type, document_number(unique), first_name, last_name, birth_date, sex, phone, email, address, emergency_contact, notes`
* **medical_specialties**: `id, name, code`
* **doctors**: `id, user_id(nullable), cmp, specialty_id(FK), phone, email, schedule_notes`
* **insurers**: `id, name, ruc, contact, phone`
* **plans**: `id, insurer_id(FK), name, coverage_rules, copay_percent`
* **admissions**: `id, patient_id, doctor_id(nullable), insurer_id(nullable), plan_id(nullable), admission_date, reason, triage_level, status(open/closed/cancelled)`
* **appointments**: `id, patient_id, doctor_id, specialty_id, scheduled_at(datetime), status(scheduled/completed/no_show/cancelled), notes`
* **audits**: `id, user_id, action, entity_type, entity_id, payload(json)`

### Índices y Relaciones

* Unicidad: `patients(document_type, document_number)`
* Índices por `doctor_id`, `patient_id`, `scheduled_at`, `status`
* Cascadas: `restrictOnDelete()` en catálogos; `cascadeOnDelete()` sólo donde sea seguro

**Ejemplo de migración (fragmento):**

```php
Schema::create('patients', function (Blueprint $table) {
    $table->id();
    $table->string('document_type', 8);
    $table->string('document_number', 32);
    $table->string('first_name');
    $table->string('last_name');
    $table->date('birth_date')->nullable();
    $table->enum('sex', ['M','F','O'])->nullable();
    $table->string('phone')->nullable();
    $table->string('email')->nullable();
    $table->string('address')->nullable();
    $table->string('emergency_contact')->nullable();
    $table->text('notes')->nullable();
    $table->timestamps();
    $table->softDeletes();

    $table->unique(['document_type', 'document_number']);
});
```

### Seeds mínimas

* Roles: `admin`, `recepcion`, `medico`
* Usuarios demo (contraseñas seguras)
* Especialidades base (medicina general, pediatría, ginecología, ...)
* Aseguradoras y planes de ejemplo
* Pacientes y citas de prueba

---

## Seguridad y Autenticación

* **Contraseñas:** `bcrypt` (`Hash::make`); validación con `Hash::check($plain, $hash)`
* **Sanctum:** Personal Access Tokens con **scopes** por rol/permiso
* **CSRF:** aplica al frontend; API con token **sin** CSRF
* **CORS multi-origen:** listar explícitamente dominios permitidos (local/prod)
* **Rate limiting:** endurecido en `/auth/*` y `/appointments`
* **Auditoría:** registrar altas/bajas/cambios críticos con `user_id` e IP
* **Backups:** dumps diarios MySQL + `.env` protegido (no versionar)

**Ejemplo `config/cors.php`:**

```php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['GET','POST','PUT','PATCH','DELETE','OPTIONS'],
    'allowed_origins' => [
        'http://localhost',
        'http://127.0.0.1:8000',
        'https://front.clinica.tld',
    ],
    'allowed_headers' => ['Content-Type','Authorization','X-Requested-With','Idempotency-Key'],
    'exposed_headers' => [],
    'max_age' => 3600,
    'supports_credentials' => false,
];
```

!!! warning "Buenas prácticas"
- Nunca almacenar ni loggear contraseñas o tokens
- Mantener `.env` fuera de Git; usar `.env.example` sanitizado
- Escapar salida en vistas (`{{ }}`) y sanitizar entradas

---

## Módulos Existentes (MVP)

1. **Autenticación y Usuarios**

   * Login/Logout, gestión de usuarios y roles
   * Emisión/rotación de tokens Sanctum

2. **Pacientes**

   * CRUD, búsqueda por documento/nombre
   * Exportaciones CSV/Excel

3. **Médicos y Especialidades**

   * CRUD, asociación a especialidades

4. **Convenios (Aseguradoras) y Planes**

   * CRUD
   * **Filtro por `insurer_id`** para planes

5. **Admisión**

   * Apertura de admisión, vínculo a paciente/aseguradora/plan, estados

6. **Citas**

   * Agendamiento simple, reprogramación
   * Estados: `scheduled`, `completed`, `no_show`, `cancelled`

7. **Dashboard**

   * KPIs: citas del día, admisiones abiertas, no_show

8. **Reportes**

   * Listados con DataTables (server-side cuando aplique)

---

## Flujo de Usuario

### Recepcionista

1. Inicia sesión → vista “Hoy”
2. Busca **Paciente** → crea/actualiza
3. Verifica **Convenio/Plan**
4. Crea **Cita** o abre **Admisión**
5. Entrega comprobante/confirmación

### Médico

1. Inicia sesión → ve **agenda** del día
2. Abre cita/admisión → registra notas
3. Actualiza estado de **Cita** (`completed` o `no_show`)

### Administrador

1. Mantiene **Usuarios/Roles/Permisos**
2. Gestiona **Catálogos** (especialidades, aseguradoras, planes)
3. Revisa **Dashboard** y auditoría

---

## Errores y Soluciones

| Problema                        | Causa típica                       | Solución / Acción                                                                      |
| ------------------------------- | ---------------------------------- | -------------------------------------------------------------------------------------- |
| CORS multi-origen               | Front local + prod                 | Listar orígenes en `config/cors.php`; headers/methods explícitos; `config:clear`       |
| `array_merge` error             | Mezcla de `null` con arrays        | `array_merge($a ?? [], $b ?? [])`; validar tipos                                       |
| “Desencriptar” password         | Expectativa de texto plano         | Usar `Hash::check`; nunca almacenar texto plano                                        |
| Fecha de Admisión               | Parse `dd/MM/yyyy` vs ISO          | Normalizar a ISO 8601 en front; `$casts` a `datetime` en Eloquent                      |
| Deploy en Hostinger             | Docroot/links/cache                | Docroot `public/`; `storage:link`; `.env`; `config:cache`, `route:cache`, `view:cache` |
| `cachePath`/optimize            | Cache roto tras mover carpetas     | `php artisan optimize:clear`; verificar `bootstrap/cache`                              |
| Token no enviado al API (front) | Falta `Authorization` o expiración | Interceptor global (axios/fetch) y manejo de 401                                       |

---

## Roadmap (MVP2 / MVP3)

### MVP2 (prioridad alta)

* **Historias Clínicas** (EAV simple o secciones predefinidas)
* **Caja/Facturación** básica (copagos; futura integración **SUNAT**)
* **Agenda avanzada** (bloques por médico, sobrecupo, recordatorios email/SMS)
* **Auditoría ampliada** (spatie/laravel-activitylog)
* **Permisos granulares** (Spatie Permissions, si no se usa ya)
* **DataTables server-side** por defecto en listados grandes

### MVP3 (evolutivo)

* **Laboratorio/Radiología** (órdenes, resultados, adjuntos)
* **Farmacia** (recetas, stock mínimo)
* **Interoperabilidad** (HL7/FHIR en endpoints selectos)
* **BI/Dashboard clínico** (Power BI/Metabase)
* **Notificaciones omnicanal** (SMS/WhatsApp/email, opt-in)

---

## Comentarios y Estilo de Código

* **Idioma oficial:** español (Perú/LatAm) para documentación, frontend y comentarios de código
* **PSR-12**, tipado estricto cuando aplique, `final`/`readonly` prudente
* **Nomenclatura:** snake_case (DB), camelCase (JS), Studly/Pascal (clases)
* **Controladores delgados**; lógica en **Services/Actions**; permisos en **Policies**
* **Validación:** `FormRequest`; **Resources** para salida consistente
* **Errores API:** 422 (validación), 401/403 (authz), 404 (no encontrado), 409 (conflicto), 500 (interno)
* **Migraciones:** idempotentes; `foreignId()->constrained()->restrictOnDelete()`
* **Seeds:** deterministas, sin datos sensibles
* **Tests iniciales:** Feature para auth, patients, admissions, appointments
* **Frontend (Blade):** layout único con `@yield('title')`, `@section('content')`, `@push('scripts')`
* **Accesibilidad:** `aria-*`, labels, contraste y estados de carga/toasts claros

---

## Endpoints API (MVP base)

> Prefijo global: `/api/v1`

**Auth**

```http
POST   /auth/login         → { token }
POST   /auth/logout
GET    /auth/me
```

**Usuarios/Roles** (admin)

```http
GET    /users
POST   /users
GET    /users/{id}
PUT    /users/{id}
DELETE /users/{id}
```

**Pacientes**

```http
GET    /patients?search=&page=
POST   /patients
GET    /patients/{id}
PUT    /patients/{id}
DELETE /patients/{id}
```

**Médicos / Especialidades**

```http
GET    /doctors
POST   /doctors
GET    /medical-specialties
POST   /medical-specialties
```

**Aseguradoras / Planes**

```http
GET    /insurers
POST   /insurers
GET    /insurers/{id}
PUT    /insurers/{id}
DELETE /insurers/{id}

GET    /insurers/{id}/plans     # filtro por aseguradora
POST   /plans
PUT    /plans/{id}
DELETE /plans/{id}
```

**Admisiones**

```http
GET    /admissions?status=
POST   /admissions
PUT    /admissions/{id}
PATCH  /admissions/{id}/close
```

**Citas**

```http
GET    /appointments?doctor_id=&date=
POST   /appointments
PUT    /appointments/{id}
PATCH  /appointments/{id}/status
```

---

## Comandos Útiles

### Local (desarrollo)

```bash
# dependencias
composer install
npm install
npm run build    # o: npm run dev

# preparar app
cp .env.example .env
php artisan key:generate
php artisan migrate --seed

# limpiar caches
php artisan optimize:clear
```

### Producción (Hostinger)

```bash
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan storage:link
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

!!! note "Checklist de despliegue"
- Docroot apuntando a `/public`
- Permisos correctos en `storage/` y `bootstrap/cache/`
- Variables reales en `.env` (no versionar)
- Revisar CORS y URLs del front en `.env`

---

## Anexo: Pseudocódigo de Idempotencia (POST crítico)

```pseudo
function handleCriticalPost(request):
  key = request.headers["Idempotency-Key"]
  if key is missing:
    proceed normally
  else:
    if exists idempotency_store[key]:
       return idempotency_store[key].response
    resp = process(request)
    idempotency_store[key] = { response: resp, created_at: now() }
    return resp
```

---

**Archivo sugerido:**

```
docs/01-tecnico/Prompt-Maestro-Sistema-Hospitalar.md
```

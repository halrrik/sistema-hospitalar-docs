# Prompt Maestro — Sistema Hospitalar (Contexto Técnico y Operativo)

> Proyecto real de sistema hospitalario para **clínica en Perú** (Cusco).  
> **Lengua oficial de documentación y comentarios en código:** **español técnico**.  
> Conversaciones pueden mezclar portugués/español, pero **lo escrito y la UI quedan en español**.

---

## 1) Alcance y Objetivo

Este documento centraliza:
- **Contexto técnico** (stack, arquitectura, entornos).
- **Reglas y convenciones** (nombres, ramas, commits, PRs).
- **Estructura de documentación** (MkDocs/MkDocs Material).
- **Plantillas de prompts** para usar en cada chat nuevo y para generar `.md`.
- **Modelo de datos textual base** y **endpoints principales**.
- **Checklist** de calidad antes de fusionar cambios.

Sirve como **fuente de verdad** para desarrolladores, PMs y stakeholders.

---

## 2) Stack Tecnológico

- **Backend:** Laravel 12 (PHP 8), API REST, Laravel Sanctum (auth/token).  
- **Frontend:** Laravel 12 + **AdminLTE** (Blade), sin SPA por defecto.  
- **BD:** MySQL (XAMPP en desarrollo), administración con phpMyAdmin.  
- **IDE/OS:** VS Code + Windows (PowerShell).  
- **Versionado:** Git + GitHub (repos: `back`, `front`, `docs`).  
- **Producción:** Hostinger (deploys según repos back/front).  
- **Documentación:** Repositorio `sistema-hospitalar-docs` + MkDocs Material (Pages).

---

## 3) Arquitectura (alto nivel)

- **Separación Front/Back por API REST**  
  - Front consume endpoints JSON seguros (tokens Sanctum, CORS configurado).  
  - Versionado de API recomendado: `/api/v1/...`.

- **Módulos Core (MVP1):**  
  Pacientes, Admisiones, Atenciones Médicas, Signos Vitales, Reportes PDF, Dashboard básico.

- **Módulos Planeados (MVP2+):**  
  Laboratorio/Resultados, Farmacia/Inventario, Facturación, Citas, Gestión avanzada de usuarios, Firma digital.

---

## 4) Entornos

- **Local (desarrollo):** Windows + XAMPP + phpMyAdmin, `APP_ENV=local`.  
- **Producción (Hostinger):** `APP_ENV=production`, caches y optimizaciones activas.  
- Mantener **.env** por entorno y nunca commitear secretos.

---

## 5) Convenciones de Proyecto

### 5.1 Idioma y Estilo
- **Documentación, comentarios de código, textos de UI:** español (Perú).
- Nombres de archivos Markdown: *Pascal-Case-Con-Guiones.md*.
- Mensajes de UI claros, con foco clínico y administrativo.

### 5.2 Ramas y Commits
- Ramas:  
  `feat/AAAA-MM-DD-HHMM-descripcion`, `fix/...`, `docs/...`, `refactor/...`.
- Commits: **Conventional Commits**  
  `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`  
  Ej.: `feat(api): crear endpoint POST /api/v1/admisiones`.

### 5.3 Pull Requests (PR)
- **Siempre vía PR** hacia `main` (main protegida; no push directo).  
- Mínimo **1 aprobación**; checks deben pasar.  
- Preferir **Squash and merge** (historial lineal).

---

## 6) Estructura de Documentación (MkDocs)



docs/
├─ 00-gobernanza/Manual-Actualizacion-Git.md
├─ 01-tecnico/Prompt-Maestro-Sistema-Hospitalar.md   ← (este archivo)
├─ 02-negocio/Documento-de-Negocio.md
├─ 03-mvp/Resumen-MVP1-MVP2.md
├─ 04-discovery/Jornadas-y-Mapa-de-Usuario.md
├─ 05-modelo-datos/Modelo-ER-Textual.md
├─ 06-manual-uso/Manual-Medicos-Recepcionistas.md
├─ 07-privacidad/Politica-Proteccion-de-Datos.md
└─ index.md



**MkDocs**: rutas en `nav` **sin** prefijo `docs/`.  
Usar admonitions:

```md
!!! note "Nota"
    Texto breve de contexto.
````

---

## 7) Seguridad y Privacidad (resumen operativo)

* Cumplir **Ley N° 29733 (Perú)** y buenas prácticas de protección de datos.
* **Mínimo necesario:** almacenar solo datos indispensables.
* **Consentimiento informado:** pacientes y personal.
* **Controles de acceso:** roles y permisos; registrar auditoría básica (quién, qué, cuándo).
* Cifrado de contraseñas (bcrypt), tokens Sanctum revocables.

---

## 8) Modelo de Datos — Base Textual

**Entidades centrales (MVP1):**

* **Pacientes**: datos personales y contacto.
* **Admisiones**: vínculo paciente + fecha/servicio + motivo.
* **Atenciones**: episodio clínico, diagnóstico, tratamiento, timestamps.
* **SignosVitales**: mediciones por atención.
* **Medicos**: identificación, especialidad.
* **Usuarios**: acceso (roles y permisos).

**Diagrama lógico (texto):**


Pacientes (id_paciente PK)
  1:N Admisiones (id_admision PK, id_paciente FK)
    1:N Atenciones (id_atencion PK, id_admision FK, id_medico FK)
      1:N SignosVitales (id_signo PK, id_atencion FK)

Medicos (id_medico PK)
  1:N Atenciones (id_medico FK)

Usuarios (id_usuario PK)
  N:1 Roles (id_rol FK)


**Campos (resumen ejemplo):**

* `pacientes`: id_paciente (PK), nombres, apellidos, documento (único), fecha_nacimiento, telefono, direccion, timestamps.
* `admisiones`: id_admision (PK), id_paciente (FK), fecha_admision (datetime), motivo, estado(`abierta|cerrada`), timestamps.
* `atenciones`: id_atencion (PK), id_admision (FK), id_medico (FK), historia_clinica (text), diagnostico (text), tratamiento (text), started_at, ended_at, timestamps.
* `signos_vitales`: id_signo (PK), id_atencion (FK), presion_arterial, temperatura, frecuencia_cardiaca, saturacion, timestamps.

*(Extender en `05-modelo-datos/Modelo-ER-Textual.md`.)*

---

## 9) API — Endpoints Base (sugeridos v1)

* **Auth:**
  `POST /api/v1/auth/login`, `POST /api/v1/auth/logout`, `GET /api/v1/auth/me`.

* **Pacientes:**
  `GET /api/v1/pacientes`, `POST /api/v1/pacientes`,
  `GET /api/v1/pacientes/{id}`, `PUT /api/v1/pacientes/{id}`, `DELETE /api/v1/pacientes/{id}`.

* **Admisiones:**
  `GET /api/v1/admisiones`, `POST /api/v1/admisiones`,
  `GET /api/v1/admisiones/{id}`, `PUT /api/v1/admisiones/{id}`, `PATCH /api/v1/admisiones/{id}/cerrar`.

* **Atenciones:**
  `GET /api/v1/atenciones`, `POST /api/v1/atenciones`,
  `GET /api/v1/atenciones/{id}`, `PUT /api/v1/atenciones/{id}`, `PATCH /api/v1/atenciones/{id}/cerrar`.

* **Signos Vitales:**
  `POST /api/v1/atenciones/{id}/signos`, `GET /api/v1/atenciones/{id}/signos`.

*(Ajustar a controladores/validaciones reales en `back`.)*

---

## 10) Flujo de Trabajo (Desarrollo y PR)

1. **Actualizar `main` local**

   ```bash
   git checkout main && git pull origin main
   ```
2. **Crear rama**

   ```bash
   git checkout -b feat/AAAA-MM-DD-HHMM-descripcion
   ```
3. **Desarrollar y probar** (Laravel artisan, seeds, Postman).
4. **Commit** (Conventional Commits) y **push**.
5. **PR a `main`** → pasar checks → 1 aprobación → **merge**.
6. **(Opcional) Tag/Release** para patch log claro.

---

## 11) Prompt Maestro (para iniciar cualquier chat nuevo)

> Pega esto al **comienzo de cada chat** sobre el proyecto.

```text
Este chat pertenece al proyecto **Sistema Hospitalar** (clínica en Perú).

Contexto técnico:
- Backend: Laravel 12 (PHP 8), API REST con Sanctum, MySQL.
- Frontend: Laravel 12 + AdminLTE (Blade), sin SPA por defecto.
- Comunicación: Front consume Back vía endpoints JSON.
- Entornos: Dev en Windows (XAMPP, phpMyAdmin), Prod en Hostinger.
- Documentación oficial (español): repo `sistema-hospitalar-docs` (MkDocs Material).
- Repos: `sistema-hospitalar-back`, `sistema-hospitalar-front`, `sistema-hospitalar-docs`.

Reglas:
- Todo lo escrito (docs, comentarios y UI) en **español técnico**.
- Mantén coherencia con arquitectura, modelo de datos y endpoints.
- Cuando se solicite documentación, responde con **Markdown limpio** siguiendo el formato del repo.

Necesito que respondas **siempre** dentro de este contexto, proponiendo soluciones pragmáticas y listas para producción.
```

---

## 12) Prompt Universal (para generar `.md` a partir de un chat)

> Úsalo cuando quieras **convertir un chat entero en documento** listo para `docs/`.

````text
Quiero que transformes el contenido de este chat en un documento técnico en **Markdown (.md)**,
siguiendo los estándares de `sistema-hospitalar-docs` (MkDocs Material).

Reglas:
1) Título H1 con el tema del chat.
2) Subtítulos H2/H3 organizados (Objetivo, Alcance, Pasos, Código, Tablas si aplica).
3) Incluir bloques de código con triple tilde (```bash, ```php, ```json).
4) Usar admonitions cuando ayuden:
   !!! note "Nota"
       Texto breve.
   !!! warning "Advertencia"
       Riesgos/atención.
5) Texto en **español técnico**.
6) Al final, indica el **archivo sugerido** (ruta en `docs/`), por ejemplo:
   `Archivo sugerido: docs/03-mvp/Resumen-MVP1-MVP2.md`
````

---

## 13) Sync Prompt (para “sincronizar” el contexto del chat)

> Útil cuando actualizaste docs y quieres que el chat **recuerde** el estado actual.

```text
Sincroniza este chat con la documentación oficial del proyecto **Sistema Hospitalar**.
Estructura principal (MkDocs):
- 01-tecnico/Prompt-Maestro-Sistema-Hospitalar.md
- 03-mvp/Resumen-MVP1-MVP2.md
- 04-discovery/Jornadas-y-Mapa-de-Usuario.md
- 05-modelo-datos/Modelo-ER-Textual.md
- 06-manual-uso/Manual-Medicos-Recepcionistas.md

A partir de ahora, responde respetando estas referencias y el estilo del repositorio.
```

---

## 14) Buenas Prácticas de Código (Laravel/Blade)

* Controladores RESTful, Requests para validación, Resources para respuesta JSON.
* Migrations + Seeders + Factories.
* Manejo de errores consistente (HTTP status + payload `{ error, message }`).
* **CORS** configurado para front en Hostinger (orígenes permitidos).
* Blade con **AdminLTE**, componentes reutilizables (layouts, cards, tablas).
* **Comentarios en español** y nombres coherentes con el dominio clínico.

---

## 15) Checklist de Calidad (previo al merge)

* [ ] Documentación y comentarios en **español técnico**.
* [ ] Cumple estructura de `docs/` y enlaces internos OK.
* [ ] Endpoints y modelos alineados con el modelo de datos.
* [ ] PR con descripción clara, cambios acotados.
* [ ] Tests/Build (si aplica) verdes; estilo de commits correcto.
* [ ] Si es entrega relevante, **tag/release** actualizada (patch log).

---

## 16) Notas y Roadmap (breve)

* **MVP1**: Pacientes, Admisiones, Atenciones, Signos, Reportes PDF, Dashboard básico.
* **MVP2**: Laboratorio, Farmacia, Facturación, Citas, Firma digital, Métricas de adopción.
* **Mejoras UX**: formularios más guiados, validaciones contextuales, accesibilidad.
* **Seguridad**: auditoría, rotación de tokens, políticas de retención de datos.

---

**Fin del documento — Prompt Maestro listo para uso.**

````


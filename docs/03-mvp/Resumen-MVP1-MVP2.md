# Resumen Ejecutivo — MVPs del Proyecto “Sistema Hospitalar”

## Objetivo del documento

Presentar, en formato ejecutivo y técnico, el estado de avance por MVP del **Sistema Hospitalar**, destacando entregas, aprendizajes, objetivos del próximo incremento y métricas de adopción, en línea con los estándares del repositorio `sistema-hospitalar-docs`.

!!! note "Contexto"
El sistema está orientado a una **clínica en Perú**, con **Laravel 12 + PHP 8**, **MySQL** (XAMPP para desarrollo), **AdminLTE** en el frontend (Blade), repositorio **Git**, despliegue en **Hostinger** y **Windows PowerShell** como terminal. Backend y frontend se gestionan como proyectos separados y se integran vía **API REST**.

---

## Alcance y Enfoque por MVP

* **MVP1**: Consolidación de la base operativa (admisión, pacientes, agenda, médicos, usuarios/roles y reportes básicos).
* **MVP2**: Ampliación clínica y administrativa (HCE, prescripción, convenios/aseguradoras, facturación, notificaciones) y mejoras de UX/UI para el flujo médico.

### Matriz de Módulos por Estado

| Módulo / Área                       | Estado             | Descripción breve                                              |
| ----------------------------------- | ------------------ | -------------------------------------------------------------- |
| Gestión de Pacientes                | Entregado (MVP1)   | Registro/edición, historial básico y datos clínicos iniciales. |
| Admisión y Atención Médica          | Entregado (MVP1)   | Flujo de ingreso, asignación de médico y consulta básica.      |
| Gestión de Médicos y Especialidades | Entregado (MVP1)   | CRUD de médicos, especialidades y disponibilidad.              |
| Agenda y Citas                      | Entregado (MVP1)   | Calendario con validación de horarios y estados.               |
| Usuarios, Roles y Permisos          | Entregado (MVP1)   | Autenticación por token y control de acceso (RBAC).            |
| Reportes operativos (DataTables)    | Entregado (MVP1)   | Listados con filtros, exportación e impresión.                 |
| Historia Clínica Electrónica (HCE)  | Planificado (MVP2) | Registro clínico estructurado por consulta/episodio.           |
| Prescripciones Médicas              | Planificado (MVP2) | Recetas digitalizadas y control de indicaciones.               |
| Convenios/Aseguradoras              | Planificado (MVP2) | Gestión de contratos, coberturas y autorizaciones.             |
| Facturación y Control Financiero    | Planificado (MVP2) | Emisión básica, cuentas por cobrar y conciliación inicial.     |
| Notificaciones y Alertas            | Planificado (MVP2) | Recordatorios de agenda y avisos internos.                     |
| Documentación y Estándar de API     | Mejora (MVP2)      | Respuestas JSON consistentes y endpoints documentados.         |

---

## MVP1 — Entregas Realizadas

### Módulos y Funciones

* **Gestión de Pacientes**: CRUD completo, historial resumido y datos clínicos básicos.
* **Admisión / Atención**: flujo de ingreso, derivación a médico y registro de consulta.
* **Médicos / Especialidades**: administración de perfiles, especialidades y disponibilidad.
* **Agenda / Citas**: programación, reprogramación y estados (confirmada, atendida, cancelada).
* **Seguridad**: autenticación por token, **RBAC** y permisos por rol.
* **Reportes básicos**: listados con **DataTables**, filtros, exportaciones e impresión.

### Usuarios Atendidos

* **Recepcionistas**, **Médicos** y **Administradores** internos.

### Pantallas Entregadas

* **Dashboard principal**.
* **Listados CRUD** (pacientes, médicos, agenda).
* **Formularios de registro/edición**.
* **Vista detallada del paciente**.

### Resultados Operativos

* **−35%** en el tiempo de admisión.
* **Trazabilidad** del paciente mejorada (historial consolidado).
* **Centralización** de información clínica básica.

!!! info "Ejemplo de flujo (alto nivel)"
`text
    Recepción → Registro/Verificación de Paciente → Admisión → Asignación de Médico
             → Consulta → (Opcional) Orden/Indicación → Alta → Reporte Operativo
    `

---

## MVP1 — Aprendizajes y Ajustes

* **Validaciones y manejo de errores**: alineación entre frontend (AdminLTE/Blade) y API.
* **Estandarización de layouts**: componentes y patrones visuales homogéneos en AdminLTE.
* **Separación de ambientes**: configuración clara de desarrollo vs. producción (Hostinger).
* **Optimización de consultas**: reducción de latencia y **seeds** más completos para pruebas.

!!! warning "Advertencia"
Asegurar la **consistencia de contratos de la API** (nombres de campos, códigos de estado y mensajes) para evitar regresiones en el frontend.

---

## MVP2 — Objetivos y Funcionalidades Planeadas

### Objetivos Principales

* Implementar **HCE** (Historia Clínica Electrónica) por episodio/consulta.
* Incorporar **Prescripciones** con control de indicaciones y auditoría.
* Gestionar **Convenios/Aseguradoras** (planes, coberturas, autorizaciones).
* Añadir **Facturación** y control financiero básico.
* Habilitar **Notificaciones/Alertas** para agenda y eventos clínicos.
* **Documentar** y estandarizar la **API REST** (contratos y guías de uso).

### Endpoints / API (alineamientos esperados)

```bash
# Pacientes
GET /api/pacientes
POST /api/pacientes
GET /api/pacientes/{id}
PUT /api/pacientes/{id}
DELETE /api/pacientes/{id}

# HCE (MVP2)
GET /api/pacientes/{id}/hce
POST /api/pacientes/{id}/hce

# Prescripciones (MVP2)
POST /api/consultas/{id}/prescripciones
GET  /api/consultas/{id}/prescripciones

# Convenios (MVP2)
GET /api/convenios
POST /api/convenios

# Facturación (MVP2)
POST /api/facturas
GET  /api/facturas/{id}
```

!!! note "Estándar de respuestas"
- `data`: recurso(s) solicitado(s)
- `meta`: paginación, totales, filtros
- `errors`: código, detalle y sugerencia de resolución

---

## MVP2 — Mejoras de UX/UI y Flujo Médico

* **Rediseño de pantallas clínicas** con navegación por etapas (admisión → atención → diagnóstico → alta).
* **Componentes AdminLTE** homogéneos con **iconografía médica** y **paleta por rol**.
* **Búsqueda predictiva** (autosuggest) de pacientes y médicos.
* **Optimización para tablet** en consultorios (touch-friendly, tipografías legibles).
* **Acciones rápidas**: iniciar atención desde admisión, reprogramar cita, generar indicación.

### Pseudodiagrama de Flujo Médico

```text
[Admisión]
  └─ Verifica paciente → Crea/Actualiza ficha
      └─ Agenda o Inicia Atención
          └─ [Consulta Médica]
              ├─ Anamnesis/Examen
              ├─ Diagnóstico(s)
              ├─ Prescripción/Indicaciones
              └─ Alta + Resumen de Atención (PDF/impresión)
```

---

## Indicadores de Adopción y Métricas

### Estado Actual (MVP1)

| Indicador                          | Valor Actual | Meta / Observación                |
| ---------------------------------- | ------------ | --------------------------------- |
| Usuarios activos mensuales (MAU)   | **85%**      | Personal administrativo y médico. |
| Tiempo promedio de admisión        | **2,8 min**  | Meta: **< 2,5 min**.              |
| Errores de formulario (vs. piloto) | **−40%**     | Validaciones front/API.           |
| Satisfacción del usuario interno   | **8,6 / 10** | Encuesta interna.                 |
| Disponibilidad (Hostinger)         | **99,3%**    | Monitoreo básico.                 |

### Métricas Objetivo (MVP2)

* **Cobertura HCE**: ≥ 80% de consultas registradas con HCE estructurada.
* **Tiempo de inicio de consulta**: ≤ 1 minuto desde admisión.
* **Tasa de re-trabajo clínico**: < 5% (ediciones por error).
* **Tasa de reclamaciones por facturación**: < 2% mensual.
* **NPS interno (clínico/administrativo)**: ≥ 60.

!!! info "Ejemplo (consulta de métricas operativas)"
`bash     # Filtrar admisiones por rango de fechas y estado
    GET /api/reportes/admisiones?from=2025-10-01&to=2025-10-31&estado=atendida
    `

---

## Conclusión y Próximos Pasos

El proyecto ha alcanzado una **base técnica y funcional sólida** en **MVP1**, validando el modelo operativo y la arquitectura basada en API.
El **MVP2** se enfocará en:

1. **Digitalización completa del acto médico** (HCE + prescripciones).
2. **Integración administrativa/financiera** (convenios, facturación).
3. **Mejoras de experiencia de usuario** (flujo médico y uso en tablet).

**Hitos inmediatos:**

* Alinear contratos de **API** y publicar la guía de consumo.
* Implementar **HCE** y **Prescripciones** con auditoría.
* Desplegar **notificaciones** y **KPIs** en panel operativo.
* Preparar evaluación de **interoperabilidad** (MVP3) con sistemas externos y analítica clínica.

---

**Archivo sugerido: docs/03-mvp/Resumen-MVP1-MVP2.md**

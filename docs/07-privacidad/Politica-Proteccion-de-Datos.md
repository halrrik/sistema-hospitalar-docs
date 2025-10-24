# Política de Protección de Datos Personales (Ley N° 29733) – Sistema Hospitalario

## Objetivo

Establecer los lineamientos técnicos, organizativos y legales para el tratamiento de datos personales y datos sensibles (médicos) dentro del **Sistema Hospitalario** utilizado por la **Clínica (sede Cusco)**, en cumplimiento de la **Ley N° 29733 – Ley de Protección de Datos Personales** y su **Reglamento (D.S. N° 003-2013-JUS)**.

!!! note "Ámbito"
Este documento aplica a todo el personal médico, asistencial y administrativo que accede al sistema; al equipo de desarrollo y operaciones (proveedor/terceros); y a cualquier tercero con acceso autorizado (laboratorios, aseguradoras, entidades públicas).

---

## Principios de Tratamiento de Datos Personales

1. **Legalidad**: El tratamiento se realiza conforme a la ley y a las buenas prácticas clínicas y administrativas.
2. **Consentimiento**: Todo tratamiento requiere consentimiento **previo, expreso, informado** del titular, salvo excepciones legales.
3. **Finalidad**: Los datos se recolectan con fines determinados y legítimos, vinculados a la **prestación de servicios de salud** y su gestión.
4. **Proporcionalidad**: Solo se tratan los datos **estrictamente necesarios** para cumplir la finalidad declarada.
5. **Calidad**: Los datos son **veraces, exactos y actualizados**.
6. **Seguridad**: Se implementan **medidas técnicas y organizativas** que garanticen confidencialidad, integridad y disponibilidad.
7. **Disposición de Recurso (ARCO)**: El titular puede ejercer **Acceso, Rectificación, Cancelación y Oposición**.
8. **Adecuación Territorial**: El tratamiento se realiza en Perú o bajo garantías equivalentes de protección.

---

## Catálogo de Datos Tratados

### Tipos de datos y ejemplos

| Categoría                            | Naturaleza             | Ejemplos                                                                                                | Base legal / Observación                                 |
| ------------------------------------ | ---------------------- | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| **Datos de identificación**          | Personales             | DNI/CE, nombres, apellidos, fecha de nacimiento, dirección, teléfono, correo                            | Necesarios para admisión y contacto                      |
| **Datos médicos (historia clínica)** | **Sensibles**          | diagnósticos, signos vitales, resultados de laboratorio, imágenes, alergias, medicamentos, antecedentes | Requieren medidas reforzadas; acceso restringido por rol |
| **Datos de aseguramiento**           | Personales             | aseguradora, número de póliza, copagos, convenios                                                       | Fines de facturación y cobertura                         |
| **Datos de contacto de emergencia**  | Personales             | nombre y teléfono de contacto                                                                           | Uso ante contingencias médicas                           |
| **Datos laborales (personal)**       | Personales             | cargo, credenciales, registros de capacitación                                                          | Control de acceso y responsabilidades                    |
| **Datos técnicos**                   | Personales / metadatos | IP, user-agent, timestamps, ID de sesión, tokens, logs de auditoría                                     | Seguridad, trazabilidad y cumplimiento                   |

!!! warning "Dato sensible"
Todo dato médico se considera **dato sensible**. Su tratamiento exige **consentimiento expreso** y controles de seguridad reforzados.

---

## Roles y Responsabilidades

### Clínica Cusco (Titular del tratamiento)

* Definir finalidades, políticas y protocolos internos.
* Designar **Responsable de Protección de Datos** (RPD).
* Obtener y custodiar consentimientos informados (pacientes y personal).
* Administrar **matriz de accesos por rol** y revisiones periódicas.
* Mantener **registro de incidencias** y notificar a la ANPDP según plazos.
* Asegurar contratos y **encargos de tratamiento** con terceros.

### Equipo Desarrollador / Proveedor (Encargado del tratamiento)

* Implementar y mantener **controles técnicos** (cifrado, segmentación, auditoría).
* **No** usar datos reales fuera de producción sin **anonimización** o autorización por escrito.
* Gestionar **ciclo seguro de desarrollo** (S-SDLC), revisión de vulnerabilidades y **parches**.
* Proveer **evidencias de seguridad** (reportes de pruebas, hardening, backups).
* Ajustarse a las políticas de la Clínica y a la normativa vigente.

### Usuarios del Sistema (Médicos, Enfermería, Administración)

* Tratar datos **solo** para la finalidad autorizada por rol.
* Mantener **confidencialidad** y **secreto profesional**.
* **No** extraer información fuera de los canales autorizados.
* Reportar **incidentes** o accesos indebidos de inmediato al RPD/TI.

---

## Consentimiento Informado

### Pacientes

* Requerido para el tratamiento de datos personales y **sensibles**.
* Debe incluir: **finalidad**, **base legal**, **destinatarios/terceros**, **plazo de conservación**, **derechos ARCO**, y **mecanismos de contacto**.

#### Modelo (extracto)

```text
Yo, [NOMBRE COMPLETO], DNI [NÚMERO], otorgo mi consentimiento previo, expreso e informado para el tratamiento de mis datos personales y datos sensibles por parte de la Clínica (sede Cusco), con la finalidad de brindar atención médica, gestionar mi historia clínica, coordinar prestaciones con aseguradoras y cumplir obligaciones legales. He sido informado/a de mis derechos ARCO y los canales para ejercerlos.
Firma: ____________  Fecha: __/__/____
```

### Personal médico y administrativo

* Aceptación de **cláusula de confidencialidad** y **uso legítimo** de la información.
* Autorización para el tratamiento de datos laborales y técnicos necesarios para **acceso** y **trazabilidad**.

---

## Medidas de Seguridad y Almacenamiento

### Controles técnicos mínimos

* **Autenticación y autorización por roles** (RBAC) con privilegios mínimos.
* **Cifrado de contraseñas** con `bcrypt` y rotación de credenciales privilegiadas.
* **TLS (HTTPS)** extremo a extremo (frontend ↔ API ↔ BD).
* **Logs de auditoría** inmutables con retención definida (accesos, lecturas, modificaciones).
* **Backups cifrados** y pruebas de **restauración** (DR/BCP).
* **Segmentación de ambientes** (desarrollo, pruebas, producción) y datos anonimizados fuera de producción.
* **Política de contraseñas** y MFA para perfiles críticos (administración, TI).
* **Hardening** de servidores, **actualizaciones** y **parcheo** regular.
* **Protección de endpoints** (rate limiting, CORS, CSRF en front Blade, validación de entrada).

### Procedimientos operativos (ejemplos)

```bash
# Verificación de certificados (servidor web)
openssl x509 -in /etc/ssl/certs/clinica_cusco.crt -noout -dates

# Rotación de llaves .env (Hostinger/producción)
php artisan key:generate
php artisan config:clear && php artisan cache:clear

# Respaldo en caliente (MySQL)
mysqldump -u usuario -p --single-transaction --routines --triggers base_clinica \
  | gzip > /backups/base_clinica_$(date +%F).sql.gz
```

!!! info "Separación de funciones"
El acceso a producción (infraestructura) y a datos médicos debe estar segregado de desarrollo. Cualquier soporte que implique acceso a datos sensibles requiere **autorización del RPD** y registro en el **libro de acceso excepcional**.

---

## Procedimiento de Gestión de Incidentes

1. **Detección y contención**
   Aislar sistemas afectados; revocar tokens/sesiones; bloquear credenciales comprometidas.

2. **Clasificación y evaluación**
   Identificar naturaleza de los datos (sensibles/no sensibles), alcance, impacto clínico y riesgo.

3. **Notificación interna**
   Escalar al **RPD**, TI/Seguridad y Dirección Médica.

4. **Notificación a la ANPDP**
   Dentro de un plazo máximo de **5 días hábiles** desde la detección, según criticidad y normativa.

5. **Comunicación a titulares**
   Si existe **riesgo significativo**, informar con medidas de mitigación y canales de contacto.

6. **Remediación y lecciones aprendidas**
   Aplicar parches, reforzar controles, actualizar políticas; registrar el incidente y acciones correctivas.

### Formato mínimo de registro de incidente (plantilla)

```text
ID: INC-YYYY-###
Fecha detección: __/__/____  Reportado por: [Nombre/Rol]
Descripción: [Resumen del evento]
Datos involucrados: [Categoría, volumen, sensibilidad]
Impacto: [Clínico/Legal/Operativo/Reputacional]
Acciones inmediatas: [Contención/Revocación/Parche]
Notificaciones: [ANPDP/Titulares]
Cierre y remediación: [Medidas aplicadas]
Lecciones aprendidas: [Mejoras]
```

!!! warning "Trazabilidad"
Todo incidente debe quedar **documentado** y **auditado**. La ausencia de registros impide demostrar cumplimiento.

---

## Período de Conservación y Eliminación

| Tipo de dato                           | Plazo de conservación                                           | Observaciones                                                        |
| -------------------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------- |
| **Historia clínica (dato sensible)**   | **≥ 10 años** desde la última atención                          | Sujeto a normativa sanitaria vigente y requerimientos médico-legales |
| Datos administrativos (paciente)       | Hasta **5 años** posteriores a la última prestación/facturación | Pueden ampliarse por obligaciones tributarias                        |
| Datos laborales y de acceso (personal) | Hasta **5 años** desde el cese                                  | Mantener lo necesario para responsabilidades y auditorías            |
| Logs de auditoría                      | **12–36 meses** según criticidad                                | Acorde a políticas de seguridad y análisis forense                   |

### Eliminación/Anonimización segura

* **Borrado criptográfico** o sobreescritura certificada en soportes físicos.
* **Anonimización** para analytics y entrenamiento (sin reidentificación).
* Acta de **disposición final** firmada por TI y RPD.

---

## Derechos ARCO y Canales

* **Acceso, Rectificación, Cancelación, Oposición**: gestionados por la Clínica a solicitud del titular.
* **Canal de contacto**: Mesa de partes / correo institucional del RPD.
* **Plazos**: Según la Ley N° 29733 y su Reglamento, con comunicación documentada al solicitante.

---

## Controles de Cumplimiento (Checklist)

* [ ] Registro de actividades de tratamiento actualizado.
* [ ] Matriz de roles/permisos revisada trimestralmente.
* [ ] Pruebas de restauración de backups realizadas (al menos semestrales).
* [ ] Simulacro de incidente y plan de respuesta (anual).
* [ ] Revisión de contratos con terceros y cláusulas de confidencialidad.
* [ ] Capacitación anual obligatoria en **protección de datos y secreto profesional**.

---

## Anexos (Sugeridos)

* **A1.** Formato de **Consentimiento Informado (Pacientes)**.
* **A2.** Cláusula de **Confidencialidad y Uso de Datos (Personal)**.
* **A3.** Procedimiento detallado de **Respuesta a Incidentes** (playbook).
* **A4.** Política de **Gestión de Accesos y Revisiones**.
* **A5.** Instructivo de **Anonimización para entornos no productivos**.

!!! note "Adaptación local – Cusco"
Considerar requisitos y prácticas del establecimiento de salud en Cusco (infraestructura, conectividad, proveedores locales) para la implementación de controles y continuidad operativa.

---

**Archivo sugerido: docs/07-privacidad/Politica-Proteccion-de-Datos.md**

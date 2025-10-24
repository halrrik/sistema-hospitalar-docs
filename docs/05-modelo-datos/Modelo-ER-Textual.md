# Modelo de Datos (ER Descriptivo) — Sistema Hospitalar

## Objetivo

Definir el **modelo de datos lógico** del Sistema Hospitalar para servir como contrato entre backend (Laravel 12 / PHP 8), frontend (AdminLTE/Blade) y analítica. El documento está preparado para leerse sin necesidad de phpMyAdmin, facilitando el alineamiento técnico entre equipos.

!!! note "Alcance"
Base de datos **MySQL (InnoDB, UTF8MB4)**, timestamps (`created_at`, `updated_at`) en todas las tablas y `deleted_at` (soft deletes) en entidades maestras. Convención `snake_case` para tablas y columnas, PK `id` (`BIGINT UNSIGNED AUTO_INCREMENT`) y FKs con sufijo `_id`.

---

## Contexto y Convenciones

* **Entorno:** Windows + XAMPP (desarrollo), Hostinger (producción).
* **Repositorio:** Backend y frontend separados; integración vía API.
* **Nomenclatura:**

  * Tablas y columnas `snake_case`.
  * `id` como PK numérica.
  * Llaves foráneas terminadas en `_id`.
  * Índices en campos de búsqueda frecuente (documentos, fechas, relaciones).
* **Borrado lógico:** `soft deletes` en entidades clínicas y maestras.
* **Catálogos clínicos:** CIE-10, procedimientos y medicamentos como referencias de solo lectura (inactivar > borrar).

!!! warning "Protección de datos"
Habilitar políticas de acceso por rol (RBAC) y auditoría de cambios sobre entidades sensibles (`pacientes`, `admisiones`, `atenciones_medicas`, `recetas`, `adjuntos_clinicos`).

---

## Diagrama lógico **textual** (vista jerárquica)

```
pacientes
 ├─ pacientes_contactos (N:1 pacientes)
 ├─ pacientes_direcciones (N:1 pacientes)
 ├─ pacientes_convenios (N:1 pacientes, N:1 convenios)  ← (N:N lógico)
 ├─ admisiones (N:1 pacientes)
 │   ├─ atenciones_medicas (N:1 admisiones, N:1 medicos)
 │   │   ├─ signos_vitales (N:1 atenciones_medicas)
 │   │   ├─ diagnosticos (N:1 atenciones_medicas, N:1 cie10_catalogo opc.)
 │   │   ├─ procedimientos (N:1 atenciones_medicas, N:1 procedimientos_cat opc.)
 │   │   ├─ recetas (N:1 atenciones_medicas)
 │   │   │   └─ recetas_detalle (N:1 recetas, N:1 medicamentos_cat opc.)
 │   │   ├─ notas_evolucion (N:1 atenciones_medicas, N:1 usuarios)
 │   │   └─ adjuntos_clinicos (N:1 atenciones_medicas, N:1 usuarios)
 │   └─ altas (N:1 admisiones, N:1 usuarios)
 ├─ citas (N:1 pacientes, N:1 medicos)
 └─ historiales (opcional; trazas clínicas agregadas)

medicos
 ├─ medicos_especialidades (N:1 medicos, N:1 especialidades) ← (N:N lógico)
 └─ agendas_medicas (N:1 medicos)

convenios (planes/seguros)
 └─ pacientes_convenios (N:1 convenios, N:1 pacientes)

especialidades
 └─ medicos_especialidades (N:1 especialidades, N:1 medicos)

usuarios (autenticación/operación)
 ├─ roles (N:N vía pivote o paquete)
 ├─ auditoria_registros (N:1 usuarios)
 └─ logs_acceso (N:1 usuarios)

recursos_fisicos (áreas/salas/camas)  [MVP2+]
 ├─ areas
 ├─ salas (N:1 areas)
 ├─ camas (N:1 salas)
 └─ ocupaciones_cama (N:1 camas, N:1 admisiones)

facturacion [MVP2+]
 ├─ facturas (N:1 pacientes, N:1 convenios opc.)
 └─ facturas_items (N:1 facturas)
```

---

## Descripción por tabla

### pacientes

**Propósito:** Maestro de personas atendidas.

| Campo                              | Tipo                           | Descripción                               |
| ---------------------------------- | ------------------------------ | ----------------------------------------- |
| id                                 | BIGINT AI                      | **PK**                                    |
| codigo_historia                    | VARCHAR(30) UNIQUE             | Identificador interno de historia clínica |
| tipo_documento                     | ENUM('DNI','CE','PASS','OTRO') | Tipo de documento                         |
| numero_documento                   | VARCHAR(20)                    | Nº documento (indexado)                   |
| nombres, apellidos                 | VARCHAR(120)                   | Identificación                            |
| sexo                               | ENUM('M','F','X')              | Sexo/autoidentificación                   |
| fecha_nacimiento                   | DATE                           | Fecha de nacimiento                       |
| telefono, email                    | VARCHAR                        | Contacto                                  |
| estado_civil                       | ENUM(...) NULL                 | Estado civil                              |
| grupo_sanguineo                    | ENUM(...) NULL                 | Grupo y factor                            |
| alergias                           | TEXT NULL                      | Alergias declaradas                       |
| observaciones                      | TEXT NULL                      | Notas                                     |
| created_at, updated_at, deleted_at | TIMESTAMP                      | Tiempos                                   |

**Relaciones:**
1:N con `pacientes_contactos`, `pacientes_direcciones`, `admisiones`, `citas`.
N:N con `convenios` vía `pacientes_convenios`.

---

### pacientes_contactos

**Propósito:** Contactos de emergencia/apoderados.

| Campo                  | Tipo                                      | Descripción                     |
| ---------------------- | ----------------------------------------- | ------------------------------- |
| id                     | BIGINT AI                                 | **PK**                          |
| paciente_id            | BIGINT                                    | **FK** → pacientes.id (CASCADE) |
| tipo                   | ENUM('EMERGENCIA','FAMILIAR','APODERADO') | Rol                             |
| nombre                 | VARCHAR(120)                              | Nombre                          |
| telefono, email        | VARCHAR                                   | Contacto                        |
| relacion               | VARCHAR(60) NULL                          | Parentesco                      |
| timestamps, deleted_at |                                           |                                 |

**Relación:** N:1 con `pacientes`.

---

### pacientes_direcciones

**Propósito:** Direcciones del paciente.

| Campo                             | Tipo              | Descripción                      |
| --------------------------------- | ----------------- | -------------------------------- |
| id                                | BIGINT AI         | **PK**                           |
| paciente_id                       | BIGINT            | **FK** → pacientes.id            |
| linea1, linea2                    | VARCHAR           | Dirección                        |
| distrito, provincia, departamento | VARCHAR           | División político-administrativa |
| pais                              | VARCHAR           | Por defecto 'Perú'               |
| referencia                        | VARCHAR(160) NULL | Punto de referencia              |
| predeterminada                    | TINYINT(1)        | 0/1                              |
| timestamps, deleted_at            |                   |                                  |

**Relación:** N:1 con `pacientes`.

---

### convenios

**Propósito:** Seguros/planes de salud.

| Campo                  | Tipo                                             | Descripción         |
| ---------------------- | ------------------------------------------------ | ------------------- |
| id                     | BIGINT AI                                        | **PK**              |
| codigo                 | VARCHAR(30) UNIQUE                               | Código del convenio |
| nombre                 | VARCHAR(160)                                     | Nombre              |
| tipo                   | ENUM('SEGURO_PRIVADO','EPS','PARTICULAR','OTRO') | Tipo                |
| estado                 | ENUM('ACTIVO','INACTIVO')                        | Estado              |
| observaciones          | TEXT NULL                                        | Notas               |
| timestamps, deleted_at |                                                  |                     |

**Relaciones:** N:N con `pacientes` vía `pacientes_convenios`; 1:N con `facturas` (MVP2).

---

### pacientes_convenios (pivote N:N)

| Campo                         | Tipo              | Descripción           |
| ----------------------------- | ----------------- | --------------------- |
| id                            | BIGINT AI         | **PK**                |
| paciente_id                   | BIGINT            | **FK** → pacientes.id |
| convenio_id                   | BIGINT            | **FK** → convenios.id |
| numero_poliza                 | VARCHAR(40) NULL  | Póliza                |
| plan                          | VARCHAR(60) NULL  | Plan                  |
| vigente_desde / vigente_hasta | DATE NULL         | Vigencias             |
| titular                       | VARCHAR(160) NULL | Titular distinto      |
| observaciones                 | TEXT NULL         | Notas                 |
| timestamps, deleted_at        |                   |                       |

---

### medicos

**Propósito:** Profesionales médicos.

| Campo                            | Tipo                      | Descripción              |
| -------------------------------- | ------------------------- | ------------------------ |
| id                               | BIGINT AI                 | **PK**                   |
| cmp                              | VARCHAR(20) UNIQUE        | Colegiatura (Perú)       |
| nombres, apellidos               | VARCHAR(120)              | Identificación           |
| tipo_documento, numero_documento | ENUM/VARCHAR              | Documento                |
| email, telefono                  | VARCHAR NULL              | Contacto                 |
| estado                           | ENUM('ACTIVO','INACTIVO') | Estado                   |
| firma_base64                     | TEXT NULL                 | Firma para recetas/notas |
| timestamps, deleted_at           |                           |                          |

**Relaciones:** N:N con `especialidades`; 1:N con `atenciones_medicas`, `citas`, `agendas_medicas`.

---

### especialidades

| Campo       | Tipo                      | Descripción |
| ----------- | ------------------------- | ----------- |
| id          | BIGINT AI                 | **PK**      |
| codigo      | VARCHAR(20) UNIQUE        | Código      |
| nombre      | VARCHAR(160)              | Nombre      |
| descripcion | TEXT NULL                 | Descripción |
| estado      | ENUM('ACTIVO','INACTIVO') | Estado      |
| timestamps  |                           |             |

**Relación:** N:N con `medicos` (pivote `medicos_especialidades`).

---

### medicos_especialidades (pivote N:N)

| Campo           | Tipo       | Descripción                     |
| --------------- | ---------- | ------------------------------- |
| id              | BIGINT AI  | **PK**                          |
| medico_id       | BIGINT     | **FK** → medicos.id             |
| especialidad_id | BIGINT     | **FK** → especialidades.id      |
| principal       | TINYINT(1) | Marca de especialidad principal |
| timestamps      |            |                                 |

---

### citas

**Propósito:** Agendamiento de pacientes con médicos.

| Campo           | Tipo                                                             | Descripción                |
| --------------- | ---------------------------------------------------------------- | -------------------------- |
| id              | BIGINT AI                                                        | **PK**                     |
| paciente_id     | BIGINT                                                           | **FK** → pacientes.id      |
| medico_id       | BIGINT                                                           | **FK** → medicos.id        |
| especialidad_id | BIGINT NULL                                                      | **FK** → especialidades.id |
| fecha_hora      | DATETIME                                                         | Inicio                     |
| duracion_min    | INT                                                              | 30 por defecto             |
| estado          | ENUM('PROGRAMADA','CONFIRMADA','ATENDIDA','CANCELADA','AUSENTE') | Flujo                      |
| motivo          | VARCHAR(200) NULL                                                | Motivo                     |
| observaciones   | TEXT NULL                                                        | Notas                      |
| timestamps      |                                                                  |                            |

---

### admisiones

**Propósito:** Episodios de atención (consulta, emergencia, hospitalización).

| Campo                  | Tipo                                                           | Descripción           |
| ---------------------- | -------------------------------------------------------------- | --------------------- |
| id                     | BIGINT AI                                                      | **PK**                |
| paciente_id            | BIGINT                                                         | **FK** → pacientes.id |
| tipo_admision          | ENUM('CONSULTA_EXTERNA','EMERGENCIA','HOSPITALIZACION','OTRO') | Tipo                  |
| fecha_ingreso          | DATETIME                                                       | Apertura              |
| motivo                 | VARCHAR(200) NULL                                              | Motivo                |
| estado                 | ENUM('ABIERTA','CERRADA','ANULADA')                            | Estado                |
| convenio_id            | BIGINT NULL                                                    | **FK** → convenios.id |
| usuario_crea_id        | BIGINT                                                         | **FK** → usuarios.id  |
| observaciones          | TEXT NULL                                                      | Notas                 |
| timestamps, deleted_at |                                                                |                       |

**Relaciones:** 1:N con `atenciones_medicas`, `altas`; [MVP2+] con `ocupaciones_cama`.

---

### atenciones_medicas

**Propósito:** Registro clínico de cada interacción.

| Campo                  | Tipo                                   | Descripción            |
| ---------------------- | -------------------------------------- | ---------------------- |
| id                     | BIGINT AI                              | **PK**                 |
| admision_id            | BIGINT                                 | **FK** → admisiones.id |
| medico_id              | BIGINT                                 | **FK** → medicos.id    |
| fecha_hora             | DATETIME                               | Marca temporal         |
| motivo_consulta        | TEXT NULL                              | Motivo                 |
| anamnesis              | TEXT NULL                              | Anamnesis              |
| examen_fisico          | TEXT NULL                              | Examen físico          |
| plan_manejo            | TEXT NULL                              | Plan                   |
| estado                 | ENUM('ABIERTA','FINALIZADA','ANULADA') | Estado                 |
| usuario_registra_id    | BIGINT                                 | **FK** → usuarios.id   |
| timestamps, deleted_at |                                        |                        |

**Relaciones:** 1:N con `signos_vitales`, `diagnosticos`, `procedimientos`, `recetas`, `notas_evolucion`, `adjuntos_clinicos`.

---

### signos_vitales

| Campo                                  | Tipo              | Descripción                    |
| -------------------------------------- | ----------------- | ------------------------------ |
| id                                     | BIGINT AI         | **PK**                         |
| atencion_id                            | BIGINT            | **FK** → atenciones_medicas.id |
| temperatura_c                          | DECIMAL(4,1) NULL | °C                             |
| frecuencia_cardiaca                    | INT NULL          | lpm                            |
| frecuencia_respiratoria                | INT NULL          | rpm                            |
| presion_sistolica / presion_diastolica | INT NULL          | mmHg                           |
| saturacion_o2                          | INT NULL          | %                              |
| peso_kg                                | DECIMAL(5,2) NULL | kg                             |
| talla_cm                               | DECIMAL(5,2) NULL | cm                             |
| imc                                    | DECIMAL(5,2) NULL | Calculado                      |
| fecha_hora                             | DATETIME          | Indexado                       |
| usuario_registra_id                    | BIGINT            | **FK** → usuarios.id           |
| timestamps                             |                   |                                |

---

### diagnosticos

| Campo        | Tipo                            | Descripción                    |
| ------------ | ------------------------------- | ------------------------------ |
| id           | BIGINT AI                       | **PK**                         |
| atencion_id  | BIGINT                          | **FK** → atenciones_medicas.id |
| codigo_cie10 | VARCHAR(10) NULL                | **FK** → cie10_catalogo.codigo |
| descripcion  | VARCHAR(255)                    | Texto libre si no hay catálogo |
| tipo         | ENUM('PRESUNTIVO','DEFINITIVO') | Clasificación                  |
| principal    | TINYINT(1)                      | Indicador de principal         |
| timestamps   |                                 |                                |

---

### procedimientos

| Campo         | Tipo             | Descripción                        |
| ------------- | ---------------- | ---------------------------------- |
| id            | BIGINT AI        | **PK**                             |
| atencion_id   | BIGINT           | **FK** → atenciones_medicas.id     |
| codigo        | VARCHAR(20) NULL | **FK** → procedimientos_cat.codigo |
| descripcion   | VARCHAR(255)     | Descripción                        |
| fecha_hora    | DATETIME         | Realización                        |
| observaciones | TEXT NULL        | Notas                              |
| timestamps    |                  |                                    |

---

### recetas

| Campo                  | Tipo        | Descripción                    |
| ---------------------- | ----------- | ------------------------------ |
| id                     | BIGINT AI   | **PK**                         |
| atencion_id            | BIGINT      | **FK** → atenciones_medicas.id |
| indicaciones_generales | TEXT NULL   | Indicaciones                   |
| requiere_firma         | TINYINT(1)  | 1 por defecto                  |
| firmada_por_id         | BIGINT NULL | **FK** → usuarios.id           |
| fecha_emision          | DATETIME    | Emisión                        |
| timestamps             |             |                                |

**Relación:** 1:N con `recetas_detalle`.

---

### recetas_detalle

| Campo                       | Tipo             | Descripción                      |
| --------------------------- | ---------------- | -------------------------------- |
| id                          | BIGINT AI        | **PK**                           |
| receta_id                   | BIGINT           | **FK** → recetas.id              |
| medicamento_codigo          | VARCHAR(30) NULL | **FK** → medicamentos_cat.codigo |
| medicamento_nombre          | VARCHAR(160)     | Nombre                           |
| dosis, frecuencia, duracion | VARCHAR(80)      | Posología                        |
| indicaciones                | TEXT NULL        | Notas                            |
| timestamps                  |                  |                                  |

---

### notas_evolucion

| Campo       | Tipo      | Descripción                    |
| ----------- | --------- | ------------------------------ |
| id          | BIGINT AI | **PK**                         |
| atencion_id | BIGINT    | **FK** → atenciones_medicas.id |
| usuario_id  | BIGINT    | **FK** → usuarios.id           |
| fecha_hora  | DATETIME  | Marca temporal                 |
| nota        | TEXT      | Contenido                      |
| timestamps  |           |                                |

---

### adjuntos_clinicos

| Campo       | Tipo                                | Descripción                    |
| ----------- | ----------------------------------- | ------------------------------ |
| id          | BIGINT AI                           | **PK**                         |
| atencion_id | BIGINT                              | **FK** → atenciones_medicas.id |
| usuario_id  | BIGINT                              | **FK** → usuarios.id           |
| tipo        | ENUM('IMAGEN','PDF','AUDIO','OTRO') | Tipo de archivo                |
| ruta        | VARCHAR(255)                        | Path/URL (storage)             |
| descripcion | VARCHAR(160) NULL                   | Notas                          |
| hash        | VARCHAR(64) NULL                    | Integridad                     |
| timestamps  |                                     |                                |

---

### altas

| Campo             | Tipo                                                              | Descripción            |
| ----------------- | ----------------------------------------------------------------- | ---------------------- |
| id                | BIGINT AI                                                         | **PK**                 |
| admision_id       | BIGINT                                                            | **FK** → admisiones.id |
| fecha_alta        | DATETIME                                                          | Cierre                 |
| motivo_alta       | ENUM('RECUPERADO','DERIVADO','VOLUNTARIO','FALLECIMIENTO','OTRO') | Motivo                 |
| instrucciones     | TEXT NULL                                                         | Indicaciones al alta   |
| usuario_cierra_id | BIGINT                                                            | **FK** → usuarios.id   |
| timestamps        |                                                                   |                        |

---

### usuarios / roles / usuarios_roles

**usuarios**

| Campo                  | Tipo                      | Descripción      |
| ---------------------- | ------------------------- | ---------------- |
| id                     | BIGINT AI                 | **PK**           |
| name                   | VARCHAR(120)              | Nombre a mostrar |
| email                  | VARCHAR(120) UNIQUE       | Login            |
| password               | VARCHAR(255)              | Hash             |
| estado                 | ENUM('ACTIVO','INACTIVO') | Estado           |
| last_login_at          | DATETIME NULL             | Último acceso    |
| remember_token         | VARCHAR(100) NULL         | Token            |
| timestamps, deleted_at |                           |                  |

**roles**

| Campo       | Tipo               | Descripción |
| ----------- | ------------------ | ----------- |
| id          | BIGINT AI          | **PK**      |
| name        | VARCHAR(60) UNIQUE | Rol         |
| descripcion | TEXT NULL          | Descripción |

**usuarios_roles** (pivote N:N)

| Campo      | Tipo      | Descripción          |
| ---------- | --------- | -------------------- |
| id         | BIGINT AI | **PK**               |
| usuario_id | BIGINT    | **FK** → usuarios.id |
| role_id    | BIGINT    | **FK** → roles.id    |

---

### auditoria_registros

| Campo          | Tipo                                              | Descripción          |
| -------------- | ------------------------------------------------- | -------------------- |
| id             | BIGINT AI                                         | **PK**               |
| usuario_id     | BIGINT                                            | **FK** → usuarios.id |
| tabla          | VARCHAR(120)                                      | Tabla afectada       |
| registro_id    | BIGINT                                            | ID del registro      |
| accion         | ENUM('CREAR','ACTUALIZAR','ELIMINAR','CONSULTAR') | Acción               |
| cambios        | JSON NULL                                         | Diff serializado     |
| ip, user_agent | VARCHAR                                           | Trazabilidad         |
| creado_en      | DATETIME                                          | Creación             |

---

### logs_acceso

| Campo          | Tipo        | Descripción                |
| -------------- | ----------- | -------------------------- |
| id             | BIGINT AI   | **PK**                     |
| usuario_id     | BIGINT      | **FK** → usuarios.id       |
| accion         | VARCHAR(60) | LOGIN/LOGOUT/TOKEN_REFRESH |
| ip, user_agent | VARCHAR     | Cliente                    |
| fecha_hora     | DATETIME    | Marca temporal             |

---

### agendas_medicas

| Campo                 | Tipo       | Descripción         |
| --------------------- | ---------- | ------------------- |
| id                    | BIGINT AI  | **PK**              |
| medico_id             | BIGINT     | **FK** → medicos.id |
| dia_semana            | TINYINT    | 1=Lunes … 7=Domingo |
| hora_inicio, hora_fin | TIME       | Franja              |
| duracion_slot_min     | INT        | 30 por defecto      |
| activo                | TINYINT(1) | 0/1                 |
| timestamps            |            |                     |

---

### cie10_catalogo (opcional)

| Campo       | Tipo             | Descripción |
| ----------- | ---------------- | ----------- |
| codigo      | VARCHAR(10)      | **PK**      |
| descripcion | VARCHAR(255)     | Descripción |
| capitulo    | VARCHAR(80) NULL | Capítulo    |

---

### procedimientos_cat / medicamentos_cat (opcionales)

**procedimientos_cat:** `codigo (PK)`, `descripcion`, `precio_ref DECIMAL(10,2) NULL`
**medicamentos_cat:** `codigo (PK)`, `nombre`, `presentacion`, `concentracion`, `unidad`

---

### areas / salas / camas / ocupaciones_cama [MVP2+]

**areas:** `id (PK)`, `nombre`, `codigo UNIQUE`
**salas:** `id (PK)`, `area_id (FK)`, `nombre`, `codigo`
**camas:** `id (PK)`, `sala_id (FK)`, `codigo UNIQUE`, `estado ENUM('LIBRE','OCUPADA','MANTENIMIENTO')`
**ocupaciones_cama:** `id (PK)`, `cama_id (FK)`, `admision_id (FK)`, `ingreso DATETIME`, `egreso DATETIME NULL`, `observaciones TEXT NULL`

---

### facturas / facturas_items [MVP2+]

**facturas:** `id (PK)`, `paciente_id (FK)`, `convenio_id (FK) NULL`, `fecha_emision DATETIME`, `estado ENUM('BORRADOR','EMITIDA','PAGADA','ANULADA')`, `total DECIMAL(12,2)`, `moneda CHAR(3) DEFAULT 'PEN'`
**facturas_items:** `id (PK)`, `factura_id (FK)`, `descripcion`, `cantidad DECIMAL(10,2)`, `precio_unit DECIMAL(12,2)`, `subtotal DECIMAL(12,2)`, `impuesto DECIMAL(12,2) DEFAULT 0`

---

## Relaciones clave (resumen)

* **Paciente** 1:N **Admisión** (histórico de episodios).
* **Admisión** 1:N **Atención Médica** (timeline dentro del episodio).
* **Atención Médica** 1:N **Signos Vitales**, **Diagnósticos**, **Procedimientos**, **Recetas** (y **recetas_detalle**), **Notas de Evolución**, **Adjuntos**.
* **Paciente** N:N **Convenio** vía `pacientes_convenios` con vigencias.
* **Médico** N:N **Especialidad** vía `medicos_especialidades`.
* **Paciente** 1:N **Cita**; **Médico** 1:N **Cita**.
* **Admisión** 1:N **Alta** (generalmente única).
* **Usuarios** como autores/firmantes/registradores en múltiples entidades.

---

## Entidades centrales (explicación funcional)

### Paciente

Maestro clínico. Punto de partida para admisiones, citas y facturación. `codigo_historia` permite búsquedas clínicas rápidas.

### Médico

Profesional con una o más especialidades y agenda. Genera atenciones, firma recetas y evoluciones.

### Admisión

Apertura de un episodio (consulta/emergencia/hospitalización). Agrupa todas las atenciones y define el estado del caso.

### Atención Médica

Interacción clínica específica dentro de una admisión: motivo, anamnesis, examen físico, plan. De ella dependen signos vitales, diagnósticos, procedimientos, recetas, notas y adjuntos.

### Signos Vitales

Mediciones seriadas con fecha/hora (temperatura, PA, FC, FR, SpO2, peso, talla, IMC).

### Diagnósticos

CIE-10 (o texto), clasificados en presuntivo/definitivo, con indicador de principal.

### Procedimientos

Acciones clínicas/terapéuticas (con o sin catálogo), con hora de realización.

### Recetas

Orden médica farmacológica con detalle de ítems. Opción de firma del médico.

### Citas

Agendamientos basados en `agendas_medicas`, con estados operativos y control de solapamiento.

### Convenios

Aseguradoras/planes. Relación N:N con pacientes y uso posterior en facturación.

---

## Integridad, índices y rendimiento

* **Referencial:** FKs `ON DELETE RESTRICT` por defecto; `ON DELETE CASCADE` cuando la entidad hija no tiene sentido sin el padre (p.ej., `signos_vitales`).
* **Índices recomendados:**

  * `pacientes(numero_documento)`, `pacientes(codigo_historia)`
  * `citas(medico_id, fecha_hora)`
  * `atenciones_medicas(admision_id, fecha_hora)`
  * `signos_vitales(atencion_id, fecha_hora)`
  * `diagnosticos(atencion_id, principal)`
* **Auditoría:** `auditoria_registros` para cambios; `logs_acceso` para eventos de autenticación.
* **Catálogos:** Referenciales estables (inactivar > borrar).
* **Archivos:** `adjuntos_clinicos.ruta` en Storage de Laravel; `hash` para integridad.

---

## Mapeo Eloquent (Laravel)

```php
// Paciente.php
class Paciente extends Model {
    use SoftDeletes;
    public function admisiones(){ return $this->hasMany(Admision::class); }
    public function citas(){ return $this->hasMany(Cita::class); }
    public function convenios(){ return $this->belongsToMany(Convenio::class, 'pacientes_convenios'); }
    public function contactos(){ return $this->hasMany(PacienteContacto::class); }
    public function direcciones(){ return $this->hasMany(PacienteDireccion::class); }
}

// Admision.php
class Admision extends Model {
    use SoftDeletes;
    public function paciente(){ return $this->belongsTo(Paciente::class); }
    public function atenciones(){ return $this->hasMany(AtencionMedica::class); }
    public function alta(){ return $this->hasMany(Alta::class); } // usualmente 1
}

// AtencionMedica.php
class AtencionMedica extends Model {
    use SoftDeletes;
    public function admision(){ return $this->belongsTo(Admision::class); }
    public function medico(){ return $this->belongsTo(Medico::class); }
    public function signos(){ return $this->hasMany(SignoVital::class); }
    public function diagnosticos(){ return $this->hasMany(Diagnostico::class); }
    public function procedimientos(){ return $this->hasMany(Procedimiento::class); }
    public function recetas(){ return $this->hasMany(Receta::class); }
    public function notas(){ return $this->hasMany(NotaEvolucion::class); }
    public function adjuntos(){ return $this->hasMany(AdjuntoClinico::class); }
}
```

!!! info "Scopes sugeridos"
- `Admision::abiertas()`
- `AtencionMedica::entreFechas($desde, $hasta)`
- `Cita::delMedico($medicoId)->pendientes()`
- `Paciente::whereDocumento($tipo, $numero)`

---

## Reglas mínimas de validación

* **Paciente:** `numero_documento` único por `tipo_documento`; `fecha_nacimiento ≤ hoy`.
* **Cita:** `fecha_hora` futura al crear; evitar solapamiento según `agendas_medicas`.
* **Admisión:** 0..1 admisión **ABIERTA** por paciente (regla configurable).
* **Atención:** `fecha_hora` dentro del rango de su admisión.
* **Signos vitales:** rangos plausibles; `imc` calculado si hay peso y talla.
* **Recetas:** al menos un `recetas_detalle`; si `requiere_firma=1`, validar `firmada_por_id`.

---

## Roadmap del modelo (MVP → MVP2/MVP3)

* **MVP actual:** Pacientes, Médicos, Citas, Admisiones, Atenciones, Signos Vitales, Diagnósticos, Procedimientos, Recetas (+ detalle), Notas, Adjuntos, Convenios, Usuarios/Roles básicos, Auditoría.
* **MVP2:** Recursos físicos (áreas/salas/camas/ocupaciones), Facturación, mejoras de agenda y recordatorios.
* **MVP3:** Laboratorio/imagen (órdenes/resultados), interconsultas, consentimientos, firma avanzada, interoperabilidad (FHIR/HL7), reportería.

---

## Ejemplos de migraciones (fragmentos)

```bash
php artisan make:model Paciente -m
php artisan make:model Admision -m
php artisan make:model AtencionMedica -m
```

```php
// database/migrations/xxxx_xx_xx_create_pacientes_table.php
Schema::create('pacientes', function (Blueprint $table) {
    $table->id();
    $table->string('codigo_historia', 30)->unique();
    $table->enum('tipo_documento', ['DNI','CE','PASS','OTRO']);
    $table->string('numero_documento', 20)->index();
    $table->string('nombres', 120);
    $table->string('apellidos', 120);
    $table->enum('sexo', ['M','F','X'])->nullable();
    $table->date('fecha_nacimiento')->nullable();
    $table->string('telefono', 30)->nullable();
    $table->string('email', 120)->nullable();
    $table->softDeletes();
    $table->timestamps();
});
```

```php
// database/migrations/xxxx_xx_xx_create_admisiones_table.php
Schema::create('admisiones', function (Blueprint $table) {
    $table->id();
    $table->foreignId('paciente_id')->constrained('pacientes');
    $table->enum('tipo_admision', ['CONSULTA_EXTERNA','EMERGENCIA','HOSPITALIZACION','OTRO'])->default('CONSULTA_EXTERNA');
    $table->dateTime('fecha_ingreso');
    $table->enum('estado', ['ABIERTA','CERRADA','ANULADA'])->default('ABIERTA');
    $table->foreignId('convenio_id')->nullable()->constrained('convenios');
    $table->foreignId('usuario_crea_id')->constrained('usuarios');
    $table->softDeletes();
    $table->timestamps();
});
```

```php
// database/migrations/xxxx_xx_xx_create_atenciones_medicas_table.php
Schema::create('atenciones_medicas', function (Blueprint $table) {
    $table->id();
    $table->foreignId('admision_id')->constrained('admisiones')->cascadeOnDelete();
    $table->foreignId('medico_id')->constrained('medicos');
    $table->dateTime('fecha_hora');
    $table->text('motivo_consulta')->nullable();
    $table->text('anamnesis')->nullable();
    $table->text('examen_fisico')->nullable();
    $table->text('plan_manejo')->nullable();
    $table->enum('estado', ['ABIERTA','FINALIZADA','ANULADA'])->default('ABIERTA');
    $table->foreignId('usuario_registra_id')->constrained('usuarios');
    $table->softDeletes();
    $table->timestamps();
});
```

---

## Notas operativas

!!! note "Auditoría"
Registrar altas, cierres de admisiones y firma de recetas en `auditoria_registros` con `accion` pertinente y `cambios` (JSON).

!!! info "Estrategia de índices"
Priorizar índices compuestos en `(admision_id, fecha_hora)`, `(medico_id, fecha_hora)` y `(atencion_id, fecha_hora)` para consultas temporales.

!!! warning "Adjuntos clínicos"
Validar el tamaño y tipo de archivo; almacenar únicamente referencias (`ruta`) en BD y contenidos en Storage (S3/local). Utilizar `hash` para deduplicación/validación.

---

**Archivo sugerido: docs/02-modelo-datos/Modelo-ER-Textual.md**

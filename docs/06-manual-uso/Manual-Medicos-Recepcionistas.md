# Manual de Uso – Sistema Hospitalar (Médicos y Recepcionistas)

## Objetivo

Proveer una guía práctica, clara y operativa para **médicos** y **recepcionistas** sobre el uso del *Sistema Hospitalar*, abarcando acceso, registro de pacientes, admisiones, inicio y cierre de atenciones, registro de signos vitales/diagnóstico e impresión de reportes clínicos.

## Alcance y Perfiles

* **Recepcionista**: registro y actualización de pacientes, creación de admisiones, soporte a impresión.
* **Médico**: gestión de atenciones, registro clínico (signos vitales, diagnóstico, indicaciones), emisión de reportes, cierre de atención.
* **Administrador** (referencia): gestión de usuarios, permisos y configuraciones (fuera del alcance operativo de este manual).

!!! note "Nota"
Los permisos visibles en menús y botones dependen del rol asignado por el Administrador del sistema.

---

## Acceso al sistema

### Requisitos previos

* Usuario y contraseña asignados por Administrador.
* Conexión a internet y navegador actualizado.

### Inicio de sesión

1. Abra la **URL del sistema** (proporcionada internamente).
2. Ingrese **Usuario** y **Contraseña**.
3. Haga clic en **Iniciar Sesión**.
4. Será redirigido al **Panel Principal**.

!!! warning "Advertencia"
Si olvidó su contraseña o su cuenta está bloqueada, contacte al Administrador. No comparta credenciales.

### Cierre de sesión

* Desde el menú superior, seleccione **Perfil → Cerrar Sesión**.

---

## Navegación (visión general)

La interfaz (basada en **AdminLTE**) utiliza:

* **Barra lateral**: módulos principales (Pacientes, Admisiones, Atenciones, Reportes).
* **Barra superior**: búsqueda rápida, perfil de usuario, opciones generales.
* **Área de trabajo**: listados, formularios y acciones (crear, editar, imprimir, cerrar).

!!! info "Ejemplo"
Menú lateral típico por rol:
- **Recepcionista**: Pacientes, Admisiones, Reportes.
- **Médico**: Atenciones, Pacientes (lectura), Reportes.

---

## Registro de pacientes (Recepcionista)

### Ruta

**Menú → Pacientes → Nuevo Paciente**.

### Pasos

1. Haga clic en **Agregar Paciente**.
2. Complete los campos obligatorios:

   * **Nombre completo**
   * **Documento (DNI/CE/Pasaporte)**
   * **Fecha de nacimiento**
   * **Sexo**
   * **Teléfono** y **Dirección** (si aplica)
3. Presione **Guardar**.

### Búsqueda y actualización

* Use el **buscador superior** por *nombre* o *documento*.
* Para editar, abra el paciente y utilice **Editar → Guardar**.

!!! note "Buena práctica"
Verifique si el paciente ya existe antes de crear uno nuevo para evitar duplicados.

---

## Admisión de pacientes (Recepcionista)

### Ruta

**Menú → Admisiones → Nueva Admisión**.

### Pasos

1. **Seleccione el paciente** (buscador por documento/nombre).
2. Elija el **Médico responsable** o **Especialidad**.
3. Complete:

   * **Fecha y hora de ingreso**
   * **Motivo de consulta**
   * **Tipo de atención** (consulta, emergencia, control, etc.)
4. Haga clic en **Registrar Admisión**.

### Resultado

* La admisión queda **abierta** y visible para el médico asignado en **Atenciones → En curso**.

!!! info "Ejemplo"
Admisión típica: Paciente “María P.”, Motivo “Cefalea intensa”, Tipo “Consulta”, Médico “Dr. R. L.”.

---

## Inicio de atención médica (Médico)

### Ruta

**Menú → Atenciones Médicas → En curso**.

### Pasos

1. Inicie sesión con su **usuario médico**.
2. En **En curso**, ubique la admisión del paciente.
3. Presione **Iniciar Atención**.
4. Se muestra la **ficha del paciente** (datos básicos, motivo de consulta).

!!! note "Nota"
Una vez iniciada la atención, tendrá disponibles pestañas para **Signos Vitales**, **Diagnóstico**, **Indicaciones** y **Resumen**.

---

## Registro de signos vitales y diagnóstico (Médico)

### Signos vitales

**Ruta:** Dentro de la atención → pestaña **Signos Vitales**.

**Pasos:**

1. Clic en **Agregar**.
2. Complete valores:

   * **Presión arterial**
   * **Frecuencia cardíaca**
   * **Temperatura**
   * **Saturación O₂**
   * **Peso** / **Talla** (si aplica)
3. Presione **Guardar**.

### Diagnóstico

**Ruta:** Dentro de la atención → pestaña **Diagnóstico**.

**Pasos:**

1. Ingrese **Descripción clínica**.
2. (Opcional) Seleccione **códigos CIE-10** si el sistema los provee.
3. Añada **observaciones** y **plan/indicaciones**.
4. Presione **Guardar Diagnóstico**.

!!! info "Ejemplo"
Diagnóstico: “Migraña sin aura (G43.0)”. Indicaciones: analgesia, hidratación, control en 48h.

---

## Impresión de reporte clínico (Médico o Recepcionista)

### Ruta

**Menú → Reportes / Historial** o desde la **Atención → Acciones → Imprimir**.

### Pasos

1. Seleccione **Paciente** y **Atención**.
2. Clic en **Imprimir Reporte**.
3. El sistema genera **PDF** con:

   * Datos del paciente
   * Signos vitales
   * Diagnóstico y observaciones
   * Fecha y **Nombre del médico**

!!! note "Nota"
El PDF puede **descargarse** o **enviarse a impresión**. Verifique impresora y márgenes del navegador.

---

## Cierre de atención (Médico o Recepcionista)

### Ruta

**Menú → Atenciones Médicas → En curso → [Atención] → Cerrar Atención**.

### Pasos

1. Revise que la atención tenga:

   * **Signos vitales** (si aplica)
   * **Diagnóstico** (y observaciones/indicaciones)
   * **Reporte impreso/guardado** (si se requiere)
2. Presione **Cerrar Atención**.
3. Confirme la operación.

### Resultado

* La atención pasa a estado **Finalizada** y aparece en **Historial**.
* El registro queda **no editable**.

!!! warning "Advertencia"
Si necesita corrección posterior al cierre, contacte al Administrador. Evite cerrar atenciones incompletas.

---

## Buenas prácticas y seguridad

* **No** use el botón “Atrás” del navegador en formularios; utilice **Cancelar/Volver** del sistema.
* **Guarde** antes de salir de cada pantalla.
* **Cierre sesión** al terminar su turno.
* **Proteja** datos sensibles; no comparta información clínica fuera de los canales autorizados.
* Mantenga datos de contacto del paciente **actualizados** para mejorar la continuidad asistencial.

---

## Tabla de referencia rápida

| Tarea                          | Rol           | Ruta / Acción principal                           |
| ------------------------------ | ------------- | ------------------------------------------------- |
| Crear paciente                 | Recepcionista | Pacientes → **Nuevo Paciente** → Guardar          |
| Buscar/editar paciente         | Recepcionista | Pacientes → Buscar → Editar → Guardar             |
| Crear admisión                 | Recepcionista | Admisiones → **Nueva Admisión** → Registrar       |
| Iniciar atención               | Médico        | Atenciones → **En curso** → **Iniciar Atención**  |
| Registrar signos vitales       | Médico        | Atención → **Signos Vitales** → Agregar → Guardar |
| Registrar diagnóstico          | Médico        | Atención → **Diagnóstico** → Guardar Diagnóstico  |
| Imprimir reporte clínico (PDF) | Médico/Recep. | Atención/Reportes → **Imprimir Reporte**          |
| Cerrar atención                | Médico/Recep. | Atención → **Cerrar Atención** → Confirmar        |

---

## Flujo visual (ASCII)

```text
[Recepcionista]          [Médico]
     |                      |
     v                      |
 Registrar Paciente         |
     |                      |
     v                      |
  Crear Admisión ---------->v
                            Iniciar Atención
                            |        \
                            v         \
                    Signos Vitales   Diagnóstico
                            \         /
                             v       v
                          Reporte (PDF)
                             |
                             v
                        Cerrar Atención
```

---

## Preguntas frecuentes (FAQ)

**1) No encuentro al paciente.**

* Verifique ortografía.
* Busque por **documento** en lugar de nombre.
* Confirme que el paciente fue **guardado**.

**2) No puedo imprimir el reporte.**

* Revise permisos del navegador para **pop-ups**.
* Descargue el PDF y **abra con un visor** externo.

**3) No veo la admisión en “En curso”.**

* Confirme que la admisión fue **registrada** y asignada al **médico correcto**.
* Actualice la vista (F5) si acaba de ser creada.

**4) Intenté cerrar la atención pero falta información.**

* Complete signos vitales/diagnóstico y vuelva a intentar.
* Si ya cerró por error, contacte al **Administrador**.

---

## Errores comunes y cómo evitarlos

* **Duplicar pacientes**: siempre buscar por documento antes de crear.
* **Perder cambios**: presionar **Guardar** antes de navegar.
* **Cerrar atenciones incompletas**: verificar pestañas **Signos** y **Diagnóstico**.
* **Problemas de impresión**: validar impresora, permisos de pop-up y formato de página.

!!! note "Soporte"
En caso de incidencias técnicas, documente: **usuario**, **fecha/hora**, **módulo**, **pasos realizados** y **capturas**, y eleve al soporte interno.

---

**Archivo sugerido: docs/02-manual-uso/Manual-Medicos-Recepcionistas.md**

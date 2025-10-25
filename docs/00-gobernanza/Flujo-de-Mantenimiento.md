# Flujo de Mantenimiento y Actualización General — Sistema Hospitalar

> Proyecto: **Sistema Hospitalar**  
> Versión inicial del flujo: v1.0 — Octubre 2025  
> Responsable: Richard N. Lartiga Mena

---

## 1. Objetivo

Garantizar que la documentación, código, repositorios y contexto de IA permanezcan sincronizados y actualizados, especialmente tras grandes cambios técnicos o funcionales (nuevos módulos, endpoints, migraciones, MVPs, etc.).

---

## 2. Periodicidad Recomendada

| Frecuencia | Propósito principal |
|-------------|---------------------|
| **Mensual** | Validar integridad, limpieza y consistencia de los repos. |
| **Por MVP / release** | Actualizar documentación, tags y changelog. |
| **Por cambio mayor (infra, DB, API)** | Revisión completa + sincronización IA. |

---

## 3. Checklist General de Mantenimiento

### 3.1 Repositorios (Back / Front / Docs)
- [ ] Verificar ramas activas y cerrar las obsoletas.  
- [ ] Confirmar que `main` está protegida (no push directo).  
- [ ] Revisar merges pendientes y conflictos abiertos.  
- [ ] Validar que los workflows de Actions corren correctamente.  
- [ ] Eliminar ramas locales ya fusionadas:  
  ```bash
  git fetch -p && git branch -vv && git branch -d nombre-rama
````

* [ ] Confirmar que las versiones (tags) están alineadas con el último release.

---

### 3.2 Documentación (`sistema-hospitalar-docs`)

* [ ] Actualizar **índice (docs/index.md)** si se añadieron nuevos archivos.
* [ ] Revisar que los enlaces internos funcionan.
* [ ] Actualizar los `.md` afectados por cambios recientes:

  * `03-mvp/Resumen-MVP1-MVP2.md`
  * `05-modelo-datos/Modelo-ER-Textual.md`
  * `06-manual-uso/Manual-Medicos-Recepcionistas.md`
* [ ] Validar renderizado local:

  ```bash
  mkdocs serve
  ```
* [ ] Realizar commit con mensaje claro:

  ```bash
  git commit -m "docs(mantenimiento): actualización de documentación general"
  ```
* [ ] Hacer push y revisar build del Pages:

  * URL: `https://halrrik.github.io/sistema-hospitalar-docs/`

---

### 3.3 Modelo de Datos y API

* [ ] Confirmar que las migraciones reflejan la estructura real de MySQL.
* [ ] Verificar seeds y relaciones (`id_paciente`, `id_admision`, `id_atencion`).
* [ ] Revisar endpoints documentados vs. existentes en `routes/api.php`.
* [ ] Si hay cambios relevantes:

  * [ ] Actualizar `05-modelo-datos/Modelo-ER-Textual.md`
  * [ ] Actualizar `Resumen-MVP1-MVP2.md`
  * [ ] Agregar nota en `CHANGELOG.md`:

    ```md
    ## [v0.X.Y] - YYYY-MM-DD
    ### Changed
    - Actualización de endpoints y modelo de datos.
    ```

---

### 3.4 Control de Versiones (Git / Releases)

* [ ] Confirmar tags (`git tag --list`) y crear nueva si procede:

  ```bash
  git tag v0.X.Y
  git push --tags
  ```
* [ ] Revisar **Release Drafter**:

  * PRs etiquetados con `feat`, `fix`, `docs`.
  * Publicar release draft si todo está completo.
* [ ] Registrar cambios destacados en `CHANGELOG.md`.

---

### 3.5 Sincronización con IA (ChatGPT)

* [ ] Confirmar que el *Prompt Maestro* refleja los cambios más recientes.
* [ ] Abrir el chat del proyecto y ejecutar el comando:

  ```text
  Sincroniza este chat con la documentación oficial del proyecto Sistema Hospitalar.
  ```
* [ ] Validar que la IA responde con base en las nuevas rutas, tablas o módulos.
* [ ] Si hubo cambios masivos:

  * [ ] Regenerar el archivo `Prompt-Maestro-Sistema-Hospitalar.md` (versión actualizada).
  * [ ] Actualizar el repositorio `docs` y volver a sincronizar.

---

### 3.6 Infraestructura (Hostinger)

* [ ] Verificar backups automáticos (base de datos y archivos).
* [ ] Confirmar despliegues exitosos (logs y acceso vía navegador).
* [ ] Revisar permisos de archivos (`storage`, `bootstrap/cache`).
* [ ] Probar endpoints críticos desde producción.

---

## 4. Mantenimiento tras Grandes Cambios (Actualización Completa)

Cuando el sistema sufre una reestructuración importante (nuevo módulo, refactor de BD, API o UI):

1. **Congelar código** (`git tag pre-update-vX.Y.Z`).
2. **Actualizar documentación** (`docs/` completo).
3. **Sincronizar IA** (nuevo *Prompt Maestro*).
4. **Publicar release mayor** (`vX+1.0.0`).
5. **Notificar equipo**: qué cambió, impacto y pasos de adaptación.

!!! warning "Advertencia"
Si los modelos o endpoints cambian de nombre, **la documentación y los seeds deben actualizarse el mismo día**.
No dejar desfasado el contexto del repositorio ni el Prompt Maestro.

---

## 5. Roles y Responsabilidades

| Rol                      | Responsabilidad principal                                |
| ------------------------ | -------------------------------------------------------- |
| **Mantenedor (Richard)** | Validar integridad de repositorios y documentación.      |
| **Editor de Docs**       | Actualizar contenido en `docs/` y regenerar sitio.       |
| **Revisor Técnico**      | Validar coherencia entre código y documentación.         |
| **Colaborador**          | Seguir el flujo Git estándar y mantener commits limpios. |

---

## 6. Registro de Mantenimiento

| Fecha      | Responsable     | Tipo de mantenimiento | Descripción breve                            |
| ---------- | --------------- | --------------------- | -------------------------------------------- |
| 2025-10-24 | Richard L. Mena | Inicialización        | Creación del flujo maestro de mantenimiento. |

*(Agregar nuevas filas según ejecuciones futuras.)*

---

**Fin del documento — Flujo de mantenimiento general y sincronización IA.**

````

---

### 💾 Para agregarlo:
```bash
git add docs/00-gobernanza/Flujo-de-Mantenimiento.md
git commit -m "docs(gobernanza): agregar checklist de mantenimiento general y sincronización IA"
git push
````

---


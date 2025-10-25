# Flujo de Mantenimiento Mensual — Sistema Hospitalar (Checklist)

> Proyecto: **Sistema Hospitalar**  
> Periodicidad: **Mensual** (o al cierre de un MVP)  
> Responsable: ___________________    Fecha: ____/____/_______

!!! note "Objetivo"
    Asegurar que **código, documentación, releases y contexto de IA** estén sincronizados tras un mes de cambios (o tras un MVP).

---

## 1 Preparación

- [ ] **Confirmar repositorios** (privados/públicos correctos):
  - back → `halrrik/sistema-hospitalar-back`
  - front → `halrrik/sistema-hospitalar-front`
  - docs → `halrrik/sistema-hospitalar-docs`
- [ ] **Actualizar `main` local** en cada repo:
  ```bash
  git checkout main && git pull origin main


---

## 2 Revisión de ramas y PRs

* [ ] Listar ramas activas y PRs abiertos (back/front/docs).
* [ ] **Cerrar** ramas obsoletas ya fusionadas.
* [ ] Verificar que **main** está protegida (PR obligatorio, sin push directo).

Guía rápida para limpiar ramas locales:

```bash
git fetch -p
git branch -vv
# eliminar ramas locales ya mergeadas
git branch -d <rama>
```

---

## 3) Documentación (MkDocs)

* [ ] Inventario de cambios del mes (qué módulos cambiaron).
* [ ] **Actualizar archivos afectados**:

  * [ ] `03-mvp/Resumen-MVP1-MVP2.md`
  * [ ] `05-modelo-datos/Modelo-ER-Textual.md`
  * [ ] `06-manual-uso/Manual-Medicos-Recepcionistas.md`
  * [ ] `01-tecnico/Prompt-Maestro-Sistema-Hospitalar.md` (si cambió arquitectura/reglas)
* [ ] **Validación local** (opcional):

  ```bash
  mkdocs serve
  # abrir http://127.0.0.1:8000/
  ```
* [ ] **Commit y push**:

  ```bash
  git add docs/**
  git commit -m "docs(mantenimiento): actualización mensual de documentación"
  git push
  ```

---

## 4) Modelo de Datos y Seeds

* [ ] Revisar **migraciones** vs. estado real de MySQL.
* [ ] Confirmar **relaciones** clave:

  * Pacientes → Admisiones → Atenciones → Signos Vitales
  * Médicos → Atenciones
* [ ] Verificar **seeds/factories** (coherencia con nuevas columnas).
* [ ] Documentar cambios en `05-modelo-datos/Modelo-ER-Textual.md`.

---

## 5) API (Back)

* [ ] Endpoints nuevos/modificados documentados (rutas, requests/responses).
* [ ] Códigos de estado y mensajes de error consistentes.
* [ ] CORS actualizado si cambió dominio del front/Hostinger.
* [ ] Agregar ejemplos `curl` o JSON cuando relevante.

---

## 6) Front (UI AdminLTE/Blade)

* [ ] Formularios y validaciones acordes a cambios del back.
* [ ] Textos y rótulos en **español técnico** (consistencia).
* [ ] Actualizar capturas en manual si hubo cambios visuales (`06-manual-uso/...`).

---

## 7) CI/CD y Pages

* [ ] Actions **verdes** en back/front/docs.
* [ ] **Docs publicadas** en Pages:

  * URL: `https://halrrik.github.io/sistema-hospitalar-docs/`
  * [ ] Navegación y enlaces internos correctos.
* [ ] (Opcional) Cache en workflows y tiempos de build aceptables.

---

## 8) Versionado y “Patch Log”

* [ ] PRs con etiquetas correctas (`feat`, `fix`, `docs`).
* [ ] **Release Drafter** actualizado (revisar draft).
* [ ] ¿Corresponde **nueva versión**?

  ```bash
  git tag v0.X.Y -m "Release mensual"
  git push --tags
  ```
* [ ] **CHANGELOG.md** actualizado (Added/Changed/Fixed).

---

## 9) Sincronización con IA (ChatGPT)

* [ ] Revisar si el **Prompt Maestro** requiere actualización.
* [ ] Ejecutar en el chat del proyecto:

  ```text
  Sincroniza este chat con la documentación oficial del proyecto Sistema Hospitalar.
  ```
* [ ] Validar que el chat responde conforme a **nuevas tablas/endpoints**.

---

## 10) Infraestructura (Hostinger)

* [ ] Despliegues OK en producción (logs, 200 OK en endpoints).
* [ ] Backups recientes de BD y archivos (política de retención).
* [ ] Permisos y **.env** sanos (`APP_ENV=production`, cache optimizado).
* [ ] Pruebas de **funciones críticas** (admisión/atención/reportes).

---

## 11) Decisiones y Riesgos (Registro)

Complete al cierre del mantenimiento:

|      Fecha | Área                  | Decisión / Cambio | Riesgo          | Acción / Mitigación | Responsable |
| ---------: | --------------------- | ----------------- | --------------- | ------------------- | ----------- |
| YYYY-MM-DD | Back/Front/Docs/Infra | …                 | Bajo/Medio/Alto | …                   | …           |

---

## 12) Resumen Ejecutivo (para dirección)

* **Cambios clave del mes:**

  * …
* **Impacto en operación/usuarios:**

  * …
* **Riesgos y mitigaciones:**

  * …
* **Próximos pasos (MVP/Release):**

  * …

---

## 13) Rubrica de Cumplimiento (auto-chequeo)

| Ítem                           | Estado |
| ------------------------------ | ------ |
| Ramas saneadas (sin obsoletas) | ☐      |
| Docs actualizadas y publicadas | ☐      |
| API y modelo alineados         | ☐      |
| Release/Tag creado si aplica   | ☐      |
| ChatGPT sincronizado           | ☐      |
| Producción validada            | ☐      |

---

## 14) Anexo — Comandos útiles

```bash
# Actualizar main
git checkout main && git pull origin main

# Crear rama mensual
git checkout -b docs/AAAA-MM-mantenimiento

# Commit estándar
git add .
git commit -m "chore(mantenimiento): cierre mensual"
git push -u origin docs/AAAA-MM-mantenimiento
```

---

**Fin del documento — Checklist mensual paso a paso.**

````



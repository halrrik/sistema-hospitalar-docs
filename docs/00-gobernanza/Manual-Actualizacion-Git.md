Boa! Seu conteúdo está correto, mas o arquivo ficou com quebras de formatação (blocos de código abertos, repetições e listas que perderam o markdown). Abaixo vai a **versão revisada e padronizada**, sem perder nada de informação. Pode **substituir** o arquivo por este conteúdo.

````md
# Manual de Actualización de Documentación (Git)

> Proyecto: **Sistema Hospitalar**  
> Repositorio: `sistema-hospitalar-docs`  
> Fuente única de la verdad: **este repo** (carpeta `docs/`)

---

## 1. Prerrequisitos
- Git instalado y configurado (`git --version`).
- Acceso al repo (SSH o HTTPS).
- Editor Markdown (VS Code recomendado).
- Convención de idioma: **español** en toda la documentación.

---

## 2. Estructura de ramas
- `main`: rama estable (siempre publicable).
- Ramas de trabajo:
  - `feat/...` para nuevas secciones o documentos.
  - `fix/...` para correcciones.
  - `docs/...` para cambios menores de documentación.

**Ejemplos:**
```bash
git checkout -b feat/modelo-datos-ampliacion
git checkout -b fix/typos-manual-uso
````

---

## 3. Flujo básico de trabajo

### 3.1 Actualizar `main` local

```bash
git checkout main
git pull origin main
```

### 3.2 Crear rama de trabajo

```bash
git checkout -b feat/nombre-descriptivo
```

### 3.3 Editar y validar

* Editar archivos en `docs/` (o agregar nuevos).
* Validar cambios (vista previa en VS Code).

### 3.4 Commit con convención

```bash
git add docs/03-mvp/Resumen-MVP1-MVP2.md
git commit -m "feat(mvp): agregar métricas de adopción y tabla de KPIs"
```

### 3.5 Push y Pull Request

```bash
git push -u origin feat/nombre-descriptivo
```

* Abrir PR en GitHub → base `main`.
* Completar el template de PR (objetivo, alcance, checklist).
* Solicitar revisión (mínimo 1).

### 3.6 Revisión y merge

* Cumplir checklist (sección 7).
* Realizar **Squash and merge** o **Merge commit**.
* Borrar rama si procede.

---

## 4. Convenciones de commit (Conventional Commits)

**Tipos más usados:**

* `feat:` contenido nuevo relevante.
* `fix:` correcciones.
* `docs:` cambios menores (formato, enlaces).
* `refactor:` reestructuración sin cambio de contenido.
* `chore:` mantenimiento.

**Ejemplos:**

```text
feat(discovery): añadir mapa hace/no hace para admisiones
fix(modelo-datos): corregir relación atenciones→admisiones
docs(negocio): actualizar KPIs de adopción 2025-Q4
```

---

## 5. Versionado semántico y changelog

* Etiquetas: `vMAJOR.MINOR.PATCH` (ej.: `v0.2.0`)
* Actualizar `CHANGELOG.md` con secciones: **Added / Changed / Fixed**.

**Comandos:**

```bash
git tag v0.2.0
git push --tags
```

---

## 6. Ubicaciones canónicas de documentos

Colocar documentación en `docs/` siguiendo esta jerarquía:

```
docs/
├─ 01-tecnico/Prompt-Maestro-Sistema-Hospitalar.md
├─ 02-negocio/Documento-de-Negocio.md
├─ 03-mvp/Resumen-MVP1-MVP2.md
├─ 04-discovery/Jornadas-y-Mapa-de-Usuario.md
├─ 05-modelo-datos/Modelo-ER-Textual.md
├─ 06-manual-uso/Manual-Medicos-Recepcionistas.md
├─ 07-privacidad/Politica-Proteccion-de-Datos.md
└─ 00-gobernanza/Manual-Actualizacion-Git.md   ← (este archivo)
```

Crear un índice navegable en `docs/index.md`:

```md
# Sistema Hospitalar — Índice de Documentación

- **Técnico**
  - [Prompt Maestro](01-tecnico/Prompt-Maestro-Sistema-Hospitalar.md)
- **Negocio**
  - [Documento de Negocio](02-negocio/Documento-de-Negocio.md)
- **MVP y Roadmap**
  - [Resumen MVP1/MVP2](03-mvp/Resumen-MVP1-MVP2.md)
- **Discovery**
  - [Jornadas y Mapa de Usuario](04-discovery/Jornadas-y-Mapa-de-Usuario.md)
- **Modelo de Datos**
  - [Modelo ER Textual](05-modelo-datos/Modelo-ER-Textual.md)
- **Manuales**
  - [Manual Médicos y Recepcionistas](06-manual-uso/Manual-Medicos-Recepcionistas.md)
- **Privacidad**
  - [Política de Protección de Datos](07-privacidad/Politica-Proteccion-de-Datos.md)
- **Gobernanza**
  - [Manual de Actualización (Git)](00-gobernanza/Manual-Actualizacion-Git.md)
```

---

## 7. Checklist antes del merge (PR)

* [ ] Texto en **español** y consistente.
* [ ] Títulos y estructura Markdown correctos (`#`, `##`, `###`).
* [ ] Enlaces internos funcionando.
* [ ] Tablas y listas legibles.
* [ ] `CHANGELOG.md` actualizado si aplica.
* [ ] Commit message con convención.
* [ ] Revisión realizada.

---

## 8. Comandos útiles

**Sincronizar con `main`:**

```bash
git fetch origin
git rebase origin/main
# o:
# git merge origin/main
```

**Resolver conflictos:**

```bash
# editar archivos con conflicto
git add <archivos_resueltos>
git rebase --continue   # o git commit si era merge
```

**Eliminar rama local y remota:**

```bash
git branch -d feat/nombre-descriptivo
git push origin --delete feat/nombre-descriptivo
```

**Histórico visual:**

```bash
git log --oneline --graph --decorate --all
```

---

## 9. Buenas prácticas

* Un cambio lógico por PR (pequeño y revisable).
* Nombres de archivos descriptivos en *Pascal-Case-Con-Guiones.md*.
* No mezclar temas (ej.: “negocio” y “modelo de datos” en el mismo PR).
* Coherencia con el **Prompt Maestro** y el **Índice**.
* Documentar decisiones relevantes al final de cada archivo (sección “Notas de decisión”).

---

## 10. Plantillas (recomendadas)

### 10.1 `.github/PULL_REQUEST_TEMPLATE.md`

```md
## Objetivo
Describir qué cambia y para qué.

## Alcance
- [ ] Documentos afectados:
  - [ ] 01-tecnico/Prompt-Maestro-Sistema-Hospitalar.md
  - [ ] 03-mvp/Resumen-MVP1-MVP2.md
  - [ ] 04-discovery/Jornadas-y-Mapa-de-Usuario.md
  - [ ] 05-modelo-datos/Modelo-ER-Textual.md
  - [ ] 06-manual-uso/Manual-Medicos-Recepcionistas.md
  - [ ] 07-privacidad/Politica-Proteccion-de-Datos.md
  - [ ] 00-gobernanza/Manual-Actualizacion-Git.md

## Checklist
- [ ] Escrito en **español**
- [ ] Enlaces internos verificados
- [ ] CHANGELOG actualizado si aplica
- [ ] Commits con convención
```

### 10.2 `.github/ISSUE_TEMPLATE/feat.yml`

```yml
name: Nueva sección o mejora
description: Proponer contenido nuevo o ampliación en los docs
labels: [feat, docs]
body:
  - type: textarea
    attributes:
      label: Objetivo de negocio/técnico
  - type: textarea
    attributes:
      label: Alcance (secciones/archivos)
  - type: textarea
    attributes:
      label: Criterios de aceptación
```

### 10.3 `.github/ISSUE_TEMPLATE/fix.yml`

```yml
name: Corrección de documentación
description: Error, incoherencia o desactualización en los docs
labels: [fix, docs]
body:
  - type: textarea
    attributes:
      label: Descripción del problema
  - type: textarea
    attributes:
      label: Evidencia / enlaces
  - type: input
    attributes:
      label: Archivos afectados
```

---

## 11. Registro de versiones (tagging)

Crear versión cuando haya cambios significativos:

```bash
git tag v0.1.0
git push --tags
```

Actualizar `CHANGELOG.md` bajo la fecha y la versión correspondiente.

---

## 12. Roles y responsabilidades

| Rol            | Función principal                                            |
| -------------- | ------------------------------------------------------------ |
| **Editor**     | Redacta o actualiza documentos en ramas `feat/*` o `docs/*`. |
| **Revisor**    | Valida estilo, enlaces y coherencia con el Índice.           |
| **Mantenedor** | Aprueba PR, gestiona versiones (tags) y `CHANGELOG`.         |

---

## Apéndice — README y navegación

En `README.md`, enlazar el índice:

```md
Consulte el **[Índice de Documentación](docs/index.md)** para navegar por todo el contenido.
```

---

## Dudas frecuentes

* **¿Puedo editar directo en `main`?** → No. Usa ramas y PR.
* **¿Qué pasa si no uso Conventional Commits?** → Se pierde trazabilidad.
* **¿La Wiki es obligatoria?** → No, opcional como portal; la fuente es `docs/`.

---

**Fin del documento**


```

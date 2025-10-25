# Dashboard – Métricas y Gráficos

Esta guía documenta los **3 gráficos del panel** y los **endpoints de la API** que los abastecen.  
Front: Laravel + AdminLTE (Blade).  
Back: Laravel API.

---

## 1. Endpoints de API

### 1.1 Admisiones por Estado (Donut)
- **Método:** `GET`
- **Ruta:** `/api/dashboard/admisiones-por-estado`
- **Respuesta:**
```json
{
  "total": 123,
  "items": [
    { "estado": "pendiente",    "total": 45 },
    { "estado": "en_progreso",  "total": 30 },
    { "estado": "finalizada",   "total": 40 },
    { "estado": "cancelada",    "total": 8  }
  ]
}
```


* **Notas de datos:** Si se usa catálogo de estados, el backend hace `JOIN` y devuelve el nombre legible. `total` es la suma de todos los ítems.

---

### 1.2 Pacientes por Distrito (Barras con M/F)

* **Método:** `GET`
* **Ruta:** `/api/dashboard/pacientes-por-distrito`
* **Filtro fijo:** `id_perfil = 7` (solo pacientes)
* **Respuesta:**

```json
{
  "labels": ["Cusco", "Santiago", "Wanchaq"],
  "datasets": [
    { "label": "Masculino", "data": [10, 7, 3] },
    { "label": "Femenino",  "data": [12, 5, 4] }
  ],
  "total": 41
}
```

* **Notas de datos:** Distrito proviene de `users.id_distrito → distritos.nombre` (cuando `NULL`, se etiqueta como *Sin Distrito*).

---

### 1.3 Pacientes por Convenio (Donut)

* **Método:** `GET`
* **Ruta:** `/api/dashboard/pacientes-por-convenio`
* **Respuesta:**

```json
{
  "total": 41,
  "items": [
    { "convenio": "Particular",   "total": 20 },
    { "convenio": "Essalud",      "total": 15 },
    { "convenio": "Sanidad PNP",  "total": 6  }
  ]
}
```

* **Notas de datos:** `users.id_convenio → convenios.nombre`. Cuando `NULL`, se etiqueta como *Sin convenio*.

---

## 2. Ubicación en la UI (Front)

Los 3 gráficos se renderizan en `resources/views/dashboard.blade.php`, **debajo** de “Consultas Agendadas” y **encima** de “Últimas Actualizaciones”.

* **Gráfico 1:** *Admisiones por Estado* (donut)
* **Gráfico 2:** *Pacientes por Distrito (Perfil=7)* (barras con split M/F)
* **Gráfico 3:** *Pacientes por Convenio* (donut)

**Dependencias:** Chart.js (AdminLTE plugin o CDN).
**Versión sugerida:** v2.9.x (el código del front está compatible con v2).

---

## 3. Configuración de entorno (Front)

El front consume la API a partir de `APP_API_URL`.

* `config/app.php`:

```php
'api_url' => env('APP_API_URL', env('APP_URL').'/api'),
```

* `.env` (no versionar):

```
APP_API_URL=http://localhost:8001/api
```

En el Blade del dashboard se expone:

```html
<meta name="api-base" content="{{ rtrim(config('app.api_url'), '/') }}">
```

Y los `fetch` usan `apiBase = meta[name="api-base"]`.

---

## 4. CORS (Back)

Si se usa front en `localhost:8000` y API en `localhost:8001`, habilitar CORS en el back:

* `config/cors.php`:

  * `paths`: incluir `api/*`
  * `allowed_origins`: incluir `http://localhost:8000`

Luego:

```bash
php artisan config:clear
```

---

## 5. Consideraciones de Performance

* Respuestas cacheadas por **5 minutos** (`Cache::remember`) en:

  * Admisiones por estado
  * Pacientes por distrito/sexo
  * Pacientes por convenio
* Ajustar TTL según volumen del ambiente productivo.

---

## 6. Pruebas rápidas (SQL)

```sql
-- Admisiones por estado
SELECT estado, COUNT(*) total
FROM admisiones
GROUP BY estado;

-- Pacientes por distrito y sexo (perfil=7)
SELECT COALESCE(d.nombre, 'Sin Distrito') AS distrito, p.sexo, COUNT(*) total
FROM users p
LEFT JOIN distritos d ON d.id = p.id_distrito
WHERE p.id_perfil = 7
GROUP BY COALESCE(d.nombre, 'Sin Distrito'), p.sexo;

-- Pacientes por convenio
SELECT COALESCE(c.nombre, 'Sin convenio') AS convenio, COUNT(*) total
FROM users p
LEFT JOIN convenios c ON c.id = p.id_convenio
GROUP BY convenio;
```

---

## 7. Errores comunes

* **404 desde el front:** Front llamando `/api/...` del mismo host. Solución: usar `APP_API_URL` y `meta[name="api-base"]`.
* **Gráfico no renderiza:** Chart.js no cargado o versión incompatible (v2 vs v3/v4).
* **Column not found (SQL):** alias con espacios o `GROUP BY` a alias en vez de a la expresión (`COALESCE(...)`).

---

````

---

# 2) `mkdocs.yml` (añadir la página al menú)

Exemplo de entrada no `nav` (ajuste a sua estrutura atual):

```yaml
nav:
  - Inicio: index.md
  - Guías:
      - Dashboard: guias/dashboard.md
  # ...demais itens
````

---

# 3) `CHANGELOG.md` (entrada recomendada)

Adicione no topo:

```markdown
## [0.4.0] - 2025-10-25
### Added
- Documentación del **Dashboard** con 3 gráficos:
  - Admisiones por Estado (donut).
  - Pacientes por Distrito con split M/F (barras).
  - Pacientes por Convenio (donut).
- Detalle de endpoints de API y configuración `APP_API_URL` en el front.
- Notas de CORS y pruebas SQL.
```

---

# 4) `README.md` (nota breve – opcional)

Adicionar um bloco curto:

```markdown
### Front ↔ API

El front usa `APP_API_URL` para resolver la base de la API.  
Ejemplo local:
```

APP_API_URL=[http://localhost:8001/api](http://localhost:8001/api)

```
En producción, apuntar al dominio público de la API (Hostinger).
```

---

# Comandos (branch, commit, push, PR)

```bash
# en el repo sistema-hospitalar-docs
git checkout -b docs/dashboard-charts
git add docs/guias/dashboard.md mkdocs.yml CHANGELOG.md README.md
git commit -m "docs(dashboard): métricas, endpoints y configuración de API (APP_API_URL); notas de CORS y pruebas SQL"
git push -u origin docs/dashboard-charts
```

Luego, abre el **PR** `docs/dashboard-charts → main`.

---

# Texto sugerido para el PR (copiar/pegar)

**Título:** docs: dashboard (gráficos y endpoints) + config APP_API_URL

**Resumen:**

* Añade guía del Dashboard con 3 gráficos (donut estados, barras distrito M/F, donut convenios).
* Documenta endpoints de API y contrato de respuesta.
* Explica `APP_API_URL` en el front y nota de CORS.
* Incluye consultas SQL de verificación y sección de troubleshooting.

**Cómo probar:**

1. Ejecutar front (`:8000`) y API (`:8001`).
2. Abrir `/dashboard` en el front y validar la carga de los 3 gráficos.
3. Verificar respuestas de:

   * `/api/dashboard/admisiones-por-estado`
   * `/api/dashboard/pacientes-por-distrito`
   * `/api/dashboard/pacientes-por-convenio`

**Checklist:**

* [ ] Builds del sitio de docs OK.
* [ ] Navegación (`mkdocs.yml`) muestra “Dashboard”.
* [ ] Sin cambios en archivos `.env`.
* [ ] CHANGELOG actualizado.

---

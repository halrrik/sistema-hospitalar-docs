# Changelog
Formato: Keep a Changelog — versión semántica.


## [0.4.0] - 2025-10-25
### Added
- **Guía del Dashboard — Métricas y Gráficos** (`docs/06-manual-uso/Dashboard-Metricas-y-Graficos.md`):
  - Documentación de 3 gráficos: Admisiones por Estado (donut), Pacientes por Distrito con split M/F (barras), Pacientes por Convenio (donut).
  - Contratos de respuesta de API y ejemplos JSON.
  - Notas de datos (catálogos, “Sin Distrito”, “Sin convenio”) y pruebas SQL.
  - Juntar nombre y apellido.
  - Notas de datos (catálogos, “Sin Distrito”, “Sin convenio”) y pruebas SQL.
- **Navegación MkDocs** actualizada (`mkdocs.yml`): entrada “Dashboard – Métricas y Gráficos”.
- **Notas de configuración** para `APP_API_URL` en el Front y recomendaciones de CORS (desarrollo local).

### Changed
- Ajustes menores de estructura en la sección **Manuales** del menú.
- Juntar nombre y apellido.
- iniciar admisiondesde pacientes.
- opcion de consulta y imediato.
- en listar pacientes solo aparece pacientes.
- sintomas se graba en signos vitales.
- correjido boton de signos vitales.


### Fixed
- Aclaraciones de compatibilidad de **Chart.js v2.9.x** para el Front (evita conflictos con v3+).

---

### Cómo migrar
No se requieren migraciones. Asegurar que `APP_API_URL` apunte al dominio correcto de la API en cada entorno.

[0.4.0]: https://github.com/halrrik/sistema-hospitalar-docs/releases/tag/v0.4.0
[Unreleased]: https://github.com/halrrik/sistema-hospitalar-docs/compare/v0.4.0...HEAD

## [v0.3.0] - 2025-10-24
### Added
- Nuevos flujos de mantenimiento general y mensual en `00-gobernanza/`.
- Actualización de navegación y enlaces en `mkdocs.yml` e `index.md`.

### Changed
- Índice de documentación ampliado (sección Gobernanza).

## [0.1.0] - 2025-10-24
### Added
- Estructura base de documentación
- Prompt Maestro (borrador)
- Pipeline de publicación (GitHub Pages)




# Changelog

## [Unreleased]

### Añadido

- Licencia GPL-3.0 o posterior para el código propio de la biblioteca.

### Cambiado

- La ruta de origen de `language_manager.inc` y `campaign_manager.inc` pasa a
  `addons/sourcemod/scripting/include/`. Actualiza los scripts que copien desde
  `include/` en la raíz del repositorio. Se conservan las directivas
  `#include <language_manager>` y `#include <campaign_manager>`.

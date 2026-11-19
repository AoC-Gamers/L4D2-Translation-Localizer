# L4D2 Translation Localizer

- Los includes públicos están en `addons/sourcemod/scripting/include/`.
  Conserva los nombres de `language_manager.inc` y `campaign_manager.inc` y sus
  contratos con los plugins consumidores.
- Revisa las dependencias de compilación y runtime descritas en
  [README.md](README.md). SM-Localizer y Left4DHooks son proveedores externos.
- Valida cambios con un consumidor SourcePawn y los includes reales de sus
  dependencias. Comprueba por separado en L4D2 cualquier cambio de comportamiento.
- Documenta cambios de API o instalación en `docs/` y `CHANGELOG.md`. Las
  revisiones Git identifican la versión de los includes que copia el consumidor.

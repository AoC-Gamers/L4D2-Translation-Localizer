# L4D2 Translation Localizer

Sistema completo de localización y gestión de traducciones para Left 4 Dead 2 en SourceMod.

## 📋 Descripción

Este proyecto proporciona dos librerías especializadas que trabajan en conjunto para ofrecer un sistema robusto de traducciones localizadas para plugins de Left 4 Dead 2:

- **`language_manager.inc`** - Gestión completa de idiomas y traducciones Valve
- **`campaign_manager.inc`** - Procesamiento dinámico de campañas y capítulos

## 🚀 Características Principales

### Language Manager
- ✅ **33 idiomas soportados** con enumeración completa
- ✅ **Traducciones Valve** con fallback automático a inglés
- ✅ **Normalización automática** LATAM → Spanish
- ✅ **Máscaras de bits** para verificaciones optimizadas de familias lingüísticas
- ✅ **Detección robusta** del idioma del cliente

### Campaign Manager  
- ✅ **Procesamiento dinámico** de códigos de mapa sin hardcoding
- ✅ **Traducciones contextuales** por modo de juego (COOP, VERSUS, etc.)
- ✅ **Soporte completo** para campañas oficiales y personalizadas
- ✅ **Sistema de fallback** en múltiples niveles
- ✅ **Herramientas de debug** integradas

## 📦 Uso en un plugin

1. Copia los dos archivos desde el directorio
   [`addons/sourcemod/scripting/include/`](addons/sourcemod/scripting/include/)
   de la revisión elegida al `scripting/include/` del plugin consumidor.
   ```
   addons/sourcemod/scripting/include/
   ├── language_manager.inc
   └── campaign_manager.inc
   ```

2. Incluye las librerías en tu plugin y recompílalo.
   ```sourcepawn
   #include <language_manager>
   #include <campaign_manager>
   ```

El plugin consumidor incorpora los stocks al compilar. SM-Localizer y
Left4DHooks se obtienen por separado para compilar y ejecutar las funciones que
usan sus natives.

## 🔧 Dependencias

### Requeridas
- **SourceMod 1.11+**
- **[SM-Localizer](https://github.com/dragokas/SM-Localizer/)** - Para traducciones Valve
- **[Left4DHooks](https://github.com/SilvDev/Left4DHooks)** - Para información de modo de juego

### Opcionales
- **MultiColors** - Para mensajes coloreados en chat

## 📚 Documentación Detallada

- **[Language Manager](docs/language_manager.md)** - API completa y ejemplos de uso
- **[Campaign Manager](docs/campaign_manager.md)** - Funciones de procesamiento de mapas
- **[Ejemplos de Integración](docs/examples.md)** - Casos de uso prácticos

## 🏗️ Arquitectura

```
┌─────────────────────┐    ┌─────────────────────┐
│   Tu Plugin         │    │   Localizer Ext     │
└──────────┬──────────┘    └──────────┬──────────┘
           │                          │
           ▼                          ▼
┌─────────────────────┐    ┌─────────────────────┐
│ campaign_manager    │◄───┤ language_manager    │
│                     │    │                     │
│ • Parseo de mapas   │    │ • 33 idiomas        │
│ • Claves dinámicas  │    │ • Traducciones Valve│
│ • Modos de juego    │    │ • Normalización     │
└─────────────────────┘    └─────────────────────┘
           │                          │
           ▼                          ▼
┌─────────────────────┐    ┌─────────────────────┐
│   Left4DHooks       │    │   SourceMod Lang    │
└─────────────────────┘    └─────────────────────┘
```

## 🎮 Mapas Soportados

### Campañas Oficiales
- **C1** - Dead Center
- **C2** - Dark Carnival  
- **C3** - Swamp Fever
- **C4** - Hard Rain
- **C5** - The Parish
- **C6** - The Passing
- **C7** - The Sacrifice
- **C8** - No Mercy
- **C9** - Crash Course
- **C10** - Death Toll
- **C11** - Dead Air
- **C12** - Blood Harvest
- **C13** - Cold Stream
- **C14** - The Last Stand

### Mapas Personalizados
El sistema funciona automáticamente con cualquier mapa que siga la convención `c[número]m[número]_nombre`.

## 🌍 Idiomas Soportados

| Código | Idioma | Código | Idioma |
|--------|--------|--------|--------|
| `ar` | Arabic | `ko` | Korean |
| `bg` | Bulgarian | `las` | Latin American Spanish |
| `chi` | Simplified Chinese | `lt` | Lithuanian |
| `cze` | Czech | `lv` | Latvian |
| `da` | Danish | `nl` | Dutch |
| `de` | German | `no` | Norwegian |
| `el` | Greek | `pl` | Polish |
| `en` | English | `pt` | Brazilian Portuguese |
| `es` | Spanish | `pt_p` | Portuguese |
| `fi` | Finnish | `ro` | Romanian |
| `fr` | French | `ru` | Russian |
| `he` | Hebrew | `sk` | Slovak |
| `hu` | Hungarian | `sv` | Swedish |
| `it` | Italian | `th` | Thai |
| `jp` | Japanese | `tr` | Turkish |
| `ua` | Ukrainian | `vi` | Vietnamese |
| `zho` | Traditional Chinese | | |

## ✨ Créditos

- **Autor**: lechuga
- **Organización**: AoC-Gamers

## Licencia

El código propio de L4D2 Translation Localizer se distribuye bajo la
[GPL-3.0 o posterior](LICENSE). Sus dependencias externas conservan
sus licencias.

---

**¿Necesitas ayuda?** Revisa la [documentación detallada](docs/) o abre un [issue](https://github.com/AoC-Gamers/l4d2-translation-localizer/issues) para obtener soporte.

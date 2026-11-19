# Language Manager Documentation

Sistema completo de gestión de idiomas para SourceMod con soporte para 33 idiomas y traducciones Valve.

## Descripción General

`language_manager.inc` es una librería optimizada que proporciona una API unificada para el manejo de idiomas en plugins de SourceMod, con especial énfasis en Left 4 Dead 2. Incluye soporte completo para traducciones Valve, normalización automática de idiomas y verificaciones optimizadas por familias lingüísticas.

## Arquitectura

### Enumeración LanguageID
```sourcepawn
enum LanguageID
{
    LANGID_ARABIC = 0,      // ar - arabic
    LANGID_BULGARIAN = 1,   // bg - bulgarian
    LANGID_SCHINESE = 2,    // chi - schinese (Simplified Chinese)
    // ... 33 idiomas totales
    LANGID_SIZE             // Total number of languages
}
```

### Arrays Sincronizados
- `sLanguageNames[]` - Nombres de idiomas ("english", "spanish", etc.)
- `sLanguageCodes[]` - Códigos de idiomas ("en", "es", etc.)

### Máscaras de Bits para Familias Lingüísticas
```sourcepawn
#define LANGMASK_ROMANCE_LANGUAGES   // Español, Francés, Italiano, etc.
#define LANGMASK_GERMANIC_LANGUAGES  // Inglés, Alemán, Holandés, etc.
#define LANGMASK_SLAVIC_LANGUAGES    // Ruso, Polaco, Checo, etc.
```

## API de Funciones

### Funciones de Búsqueda y Conversión

#### `Lang_GetLanguageName()`
Obtiene el nombre de un idioma desde su ID.

```sourcepawn
bool Lang_GetLanguageName(LanguageID langId, char[] sLangOut, int maxlength)
```

**Parámetros:**
- `langId` - ID del idioma (enum LanguageID)
- `sLangOut` - Buffer de salida para el nombre
- `maxlength` - Tamaño del buffer

**Retorna:** `true` si se encontró el idioma, `false` si está fuera de rango

**Ejemplo:**
```sourcepawn
char langName[32];
if (Lang_GetLanguageName(LANGID_SPANISH, langName, sizeof(langName)))
{
    PrintToServer("Idioma: %s", langName); // "spanish"
}
```

#### `Lang_GetLanguageCode()`
Obtiene el código de un idioma desde su ID.

```sourcepawn
bool Lang_GetLanguageCode(LanguageID langId, char[] sCodeOut, int maxlength)
```

**Ejemplo:**
```sourcepawn
char langCode[8];
if (Lang_GetLanguageCode(LANGID_SPANISH, langCode, sizeof(langCode)))
{
    PrintToServer("Código: %s", langCode); // "es"
}
```

#### `Lang_FindByCode()`
Busca un LanguageID por su código.

```sourcepawn
LanguageID Lang_FindByCode(const char[] sLangCode)
```

**Ejemplo:**
```sourcepawn
LanguageID langId = Lang_FindByCode("es");
if (langId != LANGID_ENGLISH) // No es fallback
{
    PrintToServer("Encontrado idioma español con ID: %d", view_as<int>(langId));
}
```

#### `Lang_FindByName()`
Busca un LanguageID por su nombre.

```sourcepawn
LanguageID Lang_FindByName(const char[] sLangName)
```

**Ejemplo:**
```sourcepawn
LanguageID langId = Lang_FindByName("spanish");
// langId será LANGID_SPANISH
```

### Funciones de Cliente

#### `Lang_GetClientLanguage()`
Obtiene el LanguageID de un cliente.

```sourcepawn
LanguageID Lang_GetClientLanguage(int client)
```

**Ejemplo:**
```sourcepawn
LanguageID clientLang = Lang_GetClientLanguage(client);
if (Lang_IsSpanish(clientLang))
{
    PrintToChat(client, "¡Hola! Hablas español.");
}
```

#### `Lang_GetSafeClientLanguage()`
**Función principal** para obtener idioma normalizado del cliente.

```sourcepawn
void Lang_GetSafeClientLanguage(int client, char[] sLangOut, int maxlength)
```

**Características:**
- Convierte "latam" → "spanish" automáticamente
- Fallback a "english" para clientes inválidos
- Compatible con sistemas de localización

**Ejemplo:**
```sourcepawn
char clientLang[32];
Lang_GetSafeClientLanguage(client, clientLang, sizeof(clientLang));
PrintToServer("Cliente %N usa: %s", client, clientLang);

// Salida: "Cliente Player usa: spanish" (incluso si era LATAM)
```

#### `Lang_IsClientLanguage()`
Verifica si un cliente usa un idioma específico.

```sourcepawn
bool Lang_IsClientLanguage(int client, LanguageID targetLang)
```

**Ejemplo:**
```sourcepawn
if (Lang_IsClientLanguage(client, LANGID_FRENCH))
{
    PrintToChat(client, "Bonjour! Vous parlez français.");
}
```

#### `Lang_IsClientSpanish()`
Verifica si un cliente usa español (incluye LATAM).

```sourcepawn
bool Lang_IsClientSpanish(int client)
```

**Ejemplo:**
```sourcepawn
if (Lang_IsClientSpanish(client))
{
    PrintToChat(client, "¡Hola! Detectamos que hablas español.");
}
```

### Funciones de Traducciones Valve

#### `Lang_GetValveTranslation()`
**Función principal** para obtener traducciones Valve con fallback automático.

```sourcepawn
bool Lang_GetValveTranslation(int client, const char[] valveKey, char[] translation, int maxlength, Localizer localizer = null)
```

**Proceso de Fallback:**
1. Intenta traducir en el idioma del cliente
2. Si falla, intenta en inglés
3. Si falla, retorna `false`

**Ejemplo:**
```sourcepawn
char kickText[64];
if (Lang_GetValveTranslation(client, "#L4D360UI_Kick", kickText, sizeof(kickText), g_Localizer))
{
    PrintToChat(client, "Acción: %s", kickText);
    // Español: "Expulsar"
    // Inglés: "Kick"
    // Francés: "Exclure"
}
```

#### `Lang_GetValveTranslationForLanguage()`
Obtiene traducción Valve para un idioma específico.

```sourcepawn
bool Lang_GetValveTranslationForLanguage(LanguageID langId, const char[] valveKey, char[] translation, int maxlength, Localizer localizer)
```

**Ejemplo:**
```sourcepawn
char frenchTranslation[64];
if (Lang_GetValveTranslationForLanguage(LANGID_FRENCH, "#L4D360UI_Spectate", frenchTranslation, sizeof(frenchTranslation), g_Localizer))
{
    PrintToServer("En francés: %s", frenchTranslation); // "Observer"
}
```

### Funciones de Normalización

#### `Lang_NormalizeLanguageName()`
Normaliza nombres de idiomas para compatibilidad.

```sourcepawn
void Lang_NormalizeLanguageName(const char[] input, char[] output, int maxlength)
```

**Conversiones:**
- `"latam"` → `"spanish"`
- Otros idiomas se mantienen igual

**Ejemplo:**
```sourcepawn
char normalized[32];
Lang_NormalizeLanguageName("latam", normalized, sizeof(normalized));
PrintToServer("Normalizado: %s", normalized); // "spanish"
```

### Funciones de Utilidad

#### `Lang_GetLocalizedTeamName()`
Obtiene nombres de equipos localizados para L4D2.

```sourcepawn
bool Lang_GetLocalizedTeamName(any team, int client, char[] buffer, int maxlength, Localizer localizer = null)
```

**Mapeo de Equipos:**
- `2` → Supervivientes
- `3` → Infectados  
- `1` → Espectadores

**Ejemplo:**
```sourcepawn
char teamName[32];
int clientTeam = GetClientTeam(client);
if (Lang_GetLocalizedTeamName(clientTeam, client, teamName, sizeof(teamName), g_Localizer))
{
    PrintToChat(client, "Tu equipo: %s", teamName);
}
```

#### `Lang_GetLanguageCount()`
Obtiene el número total de idiomas soportados.

```sourcepawn
int Lang_GetLanguageCount()
```

**Ejemplo:**
```sourcepawn
int totalLangs = Lang_GetLanguageCount();
PrintToServer("Idiomas soportados: %d", totalLangs); // 33
```

#### `Lang_IsValidLanguageID()`
Valida si un LanguageID está en rango válido.

```sourcepawn
bool Lang_IsValidLanguageID(LanguageID langId)
```

**Ejemplo:**
```sourcepawn
LanguageID userLang = view_as<LanguageID>(someValue);
if (Lang_IsValidLanguageID(userLang))
{
    // Es seguro usar este LanguageID
}
```

### Verificaciones Rápidas (Macros)

#### Idiomas Individuales
```sourcepawn
#define Lang_IsEnglish(%1)  (view_as<int>(%1) == view_as<int>(LANGID_ENGLISH))
#define Lang_IsSpanish(%1)  (view_as<int>(%1) == view_as<int>(LANGID_SPANISH))
#define Lang_IsLatam(%1)    (view_as<int>(%1) == view_as<int>(LANGID_LATAM))
#define Lang_IsFrench(%1)   (view_as<int>(%1) == view_as<int>(LANGID_FRENCH))
#define Lang_IsGerman(%1)   (view_as<int>(%1) == view_as<int>(LANGID_GERMAN))
#define Lang_IsItalian(%1)  (view_as<int>(%1) == view_as<int>(LANGID_ITALIAN))
#define Lang_IsRussian(%1)  (view_as<int>(%1) == view_as<int>(LANGID_RUSSIAN))
```

**Ejemplo:**
```sourcepawn
LanguageID clientLang = Lang_GetClientLanguage(client);
if (Lang_IsSpanish(clientLang))
{
    // Cliente usa español
}
```

### Funciones de Familias Lingüísticas

#### `Lang_IsRomanceLanguage()`
Verifica si pertenece a lenguas romances.

```sourcepawn
bool Lang_IsRomanceLanguage(LanguageID langId)
```

**Lenguas incluidas:** Español, LATAM, Francés, Italiano, Portugués, Brasileño, Rumano

#### `Lang_IsGermanicLanguage()`
Verifica si pertenece a lenguas germánicas.

```sourcepawn
bool Lang_IsGermanicLanguage(LanguageID langId)
```

**Lenguas incluidas:** Inglés, Alemán, Holandés, Sueco, Noruego, Danés

#### `Lang_IsSlavicLanguage()`
Verifica si pertenece a lenguas eslavas.

```sourcepawn
bool Lang_IsSlavicLanguage(LanguageID langId)
```

**Lenguas incluidas:** Ruso, Polaco, Checo, Eslovaco, Búlgaro, Ucraniano

#### `Lang_IsSpanishVariant()`
Verifica si es una variante del español.

```sourcepawn
bool Lang_IsSpanishVariant(LanguageID langId)
```

**Variantes:** Español, LATAM

#### `Lang_IsChineseVariant()`
Verifica si es una variante del chino.

```sourcepawn
bool Lang_IsChineseVariant(LanguageID langId)
```

**Variantes:** Chino Simplificado, Chino Tradicional

#### `Lang_IsPortugueseVariant()`
Verifica si es una variante del portugués.

```sourcepawn
bool Lang_IsPortugueseVariant(LanguageID langId)
```

**Variantes:** Portugués, Brasileño

## Ejemplos Prácticos

### Sistema de Bienvenida Multiidioma

```sourcepawn
public void OnClientPostAdminCheck(int client)
{
    char clientLang[32];
    Lang_GetSafeClientLanguage(client, clientLang, sizeof(clientLang));
    
    char welcomeMsg[128];
    char welcomeKey[] = "#L4D360UI_MainMenu_SurvivorLobby";
    
    if (Lang_GetValveTranslation(client, welcomeKey, welcomeMsg, sizeof(welcomeMsg), g_Localizer))
    {
        PrintToChat(client, "¡Bienvenido! %s", welcomeMsg);
    }
    else
    {
        PrintToChat(client, "Welcome! Language: %s", clientLang);
    }
}
```

### Sistema de Comandos por Familia Lingüística

```sourcepawn
public Action CMD_Help(int client, int args)
{
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    if (Lang_IsRomanceLanguage(clientLang))
    {
        PrintToChat(client, "Usa: !ayuda para obtener ayuda");
    }
    else if (Lang_IsGermanicLanguage(clientLang))
    {
        PrintToChat(client, "Use: !help to get help");
    }
    else if (Lang_IsSlavicLanguage(clientLang))
    {
        PrintToChat(client, "Используй: !помощь для получения помощи");
    }
    else
    {
        PrintToChat(client, "Use: !help to get help (default)");
    }
    
    return Plugin_Handled;
}
```

### Sistema de Estadísticas de Idiomas

```sourcepawn
int g_LanguageStats[LANGID_SIZE];

public void OnClientPostAdminCheck(int client)
{
    LanguageID clientLang = Lang_GetClientLanguage(client);
    g_LanguageStats[view_as<int>(clientLang)]++;
}

public Action CMD_LangStats(int client, int args)
{
    PrintToChat(client, "=== Estadísticas de Idiomas ===");
    
    for (int i = 0; i < view_as<int>(LANGID_SIZE); i++)
    {
        if (g_LanguageStats[i] > 0)
        {
            char langName[32];
            Lang_GetLanguageName(view_as<LanguageID>(i), langName, sizeof(langName));
            PrintToChat(client, "%s: %d jugadores", langName, g_LanguageStats[i]);
        }
    }
    
    return Plugin_Handled;
}
```

## Debugging y Troubleshooting

### Problemas Comunes

#### 1. Traducciones no aparecen
```sourcepawn
// Verificar si Localizer está listo
if (!g_Localizer.IsReady())
{
    LogError("Localizer no está listo. Verifica la configuración.");
    return;
}

// Verificar clave de traducción
char testKey[] = "#L4D360UI_Kick";
char result[64];
if (!Lang_GetValveTranslation(client, testKey, result, sizeof(result), g_Localizer))
{
    LogError("No se pudo traducir la clave: %s", testKey);
}
```

#### 2. Cliente con idioma inválido
```sourcepawn
LanguageID clientLang = Lang_GetClientLanguage(client);
if (!Lang_IsValidLanguageID(clientLang))
{
    LogError("Cliente %d tiene LanguageID inválido: %d", client, view_as<int>(clientLang));
    // Usar fallback
    clientLang = LANGID_ENGLISH;
}
```

### Logs de Debug

```sourcepawn
// Habilitar logging detallado
public void DebugClientLanguage(int client)
{
    char langName[32], langCode[8], safeClientLang[32];
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    Lang_GetLanguageName(clientLang, langName, sizeof(langName));
    Lang_GetLanguageCode(clientLang, langCode, sizeof(langCode));
    Lang_GetSafeClientLanguage(client, safeClientLang, sizeof(safeClientLang));
    
    LogMessage("Cliente %N:", client);
    LogMessage("  - LanguageID: %d", view_as<int>(clientLang));
    LogMessage("  - Nombre: %s", langName);
    LogMessage("  - Código: %s", langCode);
    LogMessage("  - Normalizado: %s", safeClientLang);
    LogMessage("  - Es español: %s", Lang_IsClientSpanish(client) ? "Sí" : "No");
    LogMessage("  - Familia romance: %s", Lang_IsRomanceLanguage(clientLang) ? "Sí" : "No");
}
```

## Performance y Optimización

### Mejores Prácticas

1. **Usar macros para verificaciones frecuentes:**
```sourcepawn
// Rápido (macro inline)
if (Lang_IsSpanish(clientLang)) { }

// Más lento (llamada a función)
if (Lang_IsClientLanguage(client, LANGID_SPANISH)) { }
```

2. **Cache el idioma del cliente:**
```sourcepawn
// Cache una vez
char g_ClientLang[MAXPLAYERS + 1][32];

public void OnClientPostAdminCheck(int client)
{
    Lang_GetSafeClientLanguage(client, g_ClientLang[client], sizeof(g_ClientLang[]));
}
```

3. **Usar máscaras de bits para múltiples verificaciones:**
```sourcepawn
// Una operación bit a bit
if (Lang_IsRomanceLanguage(clientLang)) { }

// Múltiples comparaciones
if (clientLang == LANGID_SPANISH || clientLang == LANGID_FRENCH || ...) { }
```

---

**Siguiente:** [Campaign Manager Documentation](campaign_manager.md)

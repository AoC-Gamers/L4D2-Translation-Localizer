# Campaign Manager Documentation

Sistema dinámico de gestión de campañas y capítulos para Left 4 Dead 2 con traducciones localizadas.

## 📋 Descripción General

`campaign_manager.inc` es una librería especializada que procesa códigos de mapas de Left 4 Dead 2 para generar nombres localizados de campañas y capítulos. Utiliza un sistema dinámico sin hardcoding que funciona con campañas oficiales y personalizadas, generando automáticamente las claves de traducción Valve correspondientes.

## 🏗️ Arquitectura

### Constantes y Definiciones
```sourcepawn
#define MAX_MAP_CODE_LENGTH        32
#define MAX_CAMPAIGN_NAME_LENGTH   64
#define MAX_CHAPTER_NAME_LENGTH    128
#define MAP_PREFIX_L4D2            "L4D2"
```

### Estructura de Datos
```sourcepawn
enum struct ChapterInfo
{
    char mapCode[MAX_MAP_CODE_LENGTH];      // "c1m1_hotel"
    char displayName[MAX_CHAPTER_NAME_LENGTH]; // Fallback name
    char translationKey[64];                // "#L4D360UI_LevelName_COOP_c1m1_hotel"
}
```

### Pipeline de Procesamiento
```
Input: "L4D2C5M4_quarter"
  ↓ Campaign_RemoveMapPrefix()
"C5M4_quarter"  
  ↓ Campaign_ExtractCampaignCode()
"c5"
  ↓ Campaign_GenerateTranslationKey()
"#L4D360UI_CampaignName_C5"
```

## 🔧 API de Funciones

### Funciones de Parsing de Códigos

#### `Campaign_RemoveMapPrefix()`
Elimina prefijos comunes de L4D2 de un código de mapa.

```sourcepawn
bool Campaign_RemoveMapPrefix(const char[] mapCode, char[] output, int maxlength)
```

**Parámetros:**
- `mapCode` - Código de entrada ("L4D2C1", "l4d2c5m1")
- `output` - Buffer de salida para código limpio
- `maxlength` - Tamaño del buffer de salida

**Retorna:** `true` si se eliminó un prefijo, `false` en caso contrario

**Ejemplos:**
```sourcepawn
char cleanCode[32];

// Ejemplo 1: Prefijo L4D2 en mayúsculas
Campaign_RemoveMapPrefix("L4D2C1M1", cleanCode, sizeof(cleanCode));
// Resultado: cleanCode = "C1M1"

// Ejemplo 2: Prefijo l4d2 en minúsculas  
Campaign_RemoveMapPrefix("l4d2c5m4_quarter", cleanCode, sizeof(cleanCode));
// Resultado: cleanCode = "c5m4_quarter"

// Ejemplo 3: Sin prefijo
Campaign_RemoveMapPrefix("c1m1_hotel", cleanCode, sizeof(cleanCode));  
// Resultado: cleanCode = "c1m1_hotel"
```

#### `Campaign_ExtractCampaignCode()`
Extrae el código de campaña de un código de mapa completo.

```sourcepawn
bool Campaign_ExtractCampaignCode(const char[] mapCode, char[] campaignCode, int maxlength)
```

**Parámetros:**
- `mapCode` - Código de mapa de entrada
- `campaignCode` - Buffer de salida para código de campaña  
- `maxlength` - Tamaño del buffer de salida

**Retorna:** `true` si se extrajo correctamente, `false` si el formato es inválido

**Ejemplos:**
```sourcepawn
char campaignCode[8];

// Campañas oficiales
Campaign_ExtractCampaignCode("c1m1_hotel", campaignCode, sizeof(campaignCode));
// Resultado: campaignCode = "c1"

Campaign_ExtractCampaignCode("C5M4_quarter", campaignCode, sizeof(campaignCode));  
// Resultado: campaignCode = "c5" (convertido a minúsculas)

// Campañas personalizadas
Campaign_ExtractCampaignCode("c14m3_custommap", campaignCode, sizeof(campaignCode));
// Resultado: campaignCode = "c14"

// Con prefijo L4D2
Campaign_ExtractCampaignCode("L4D2C8M1_greenhouse", campaignCode, sizeof(campaignCode));
// Resultado: campaignCode = "c8"
```

#### `Campaign_ExtractMapCode()`  
Extrae el código completo del mapa (campaña + capítulo).

```sourcepawn
bool Campaign_ExtractMapCode(const char[] mapName, char[] mapCode, int maxlength)
```

**Parámetros:**
- `mapName` - Nombre completo del mapa
- `mapCode` - Buffer de salida para código de mapa
- `maxlength` - Tamaño del buffer de salida

**Retorna:** `true` si se extrajo correctamente, `false` si el formato es inválido

**Ejemplos:**
```sourcepawn
char mapCode[16];

// Extraer código completo
Campaign_ExtractMapCode("c1m1_hotel", mapCode, sizeof(mapCode));
// Resultado: mapCode = "c1m1"

Campaign_ExtractMapCode("C5M4_quarter", mapCode, sizeof(mapCode));
// Resultado: mapCode = "c5m4"

Campaign_ExtractMapCode("L4D2C8M1_greenhouse", mapCode, sizeof(mapCode));  
// Resultado: mapCode = "c8m1"
```

### Funciones de Generación de Claves

#### `Campaign_GenerateTranslationKey()`
Genera dinámicamente una clave de traducción Valve para campaña.

```sourcepawn
bool Campaign_GenerateTranslationKey(const char[] campaignCode, char[] buffer, int maxlength)
```

**Parámetros:**
- `campaignCode` - Código de campaña ("c1", "c5", "c10")
- `buffer` - Buffer de salida para clave de traducción
- `maxlength` - Tamaño del buffer de salida

**Retorna:** `true` si se generó la clave, `false` si el código es inválido

**Ejemplos:**
```sourcepawn
char translationKey[64];

// Campañas oficiales
Campaign_GenerateTranslationKey("c1", translationKey, sizeof(translationKey));
// Resultado: "#L4D360UI_CampaignName_C1"

Campaign_GenerateTranslationKey("c5", translationKey, sizeof(translationKey));
// Resultado: "#L4D360UI_CampaignName_C5"

// Campañas DLC/personalizadas
Campaign_GenerateTranslationKey("c14", translationKey, sizeof(translationKey));
// Resultado: "#L4D360UI_CampaignName_C14"

// Código inválido
Campaign_GenerateTranslationKey("invalid", translationKey, sizeof(translationKey));
// Resultado: false, buffer vacío
```

### Funciones de Localización de Campañas

#### `Campaign_GetLocalizedName()`
Obtiene el nombre localizado de una campaña usando generación dinámica de claves.

```sourcepawn
bool Campaign_GetLocalizedName(const char[] campaignCode, int client, char[] buffer, int maxlength, Localizer localizer = null)
```

**Parámetros:**
- `campaignCode` - Código de campaña ("c1", "c5")
- `client` - Índice del cliente para detección de idioma
- `buffer` - Buffer de salida para nombre localizado
- `maxlength` - Tamaño del buffer de salida
- `localizer` - Instancia de Localizer para traducciones

**Retorna:** `true` si se encontró traducción localizada, `false` si usa fallback

**Sistema de Fallback:**
1. Traducción Valve en idioma del cliente
2. Código de campaña original
3. "Unknown Campaign" si el código es inválido

**Ejemplos:**
```sourcepawn
char campaignName[64];

// Cliente español
if (Campaign_GetLocalizedName("c1", client, campaignName, sizeof(campaignName), g_Localizer))
{
    PrintToChat(client, "Campaña: %s", campaignName); 
    // Resultado: "Centro Urbano"
}

// Cliente inglés
if (Campaign_GetLocalizedName("c5", client, campaignName, sizeof(campaignName), g_Localizer))
{
    PrintToChat(client, "Campaign: %s", campaignName);
    // Resultado: "The Parish"  
}

// Campaña personalizada sin traducción
Campaign_GetLocalizedName("c99", client, campaignName, sizeof(campaignName), g_Localizer);
// Resultado: campaignName = "c99" (fallback)
```

#### `Campaign_GetLocalizedNameFromMapCode()`
Obtiene el nombre localizado de campaña directamente desde un código de mapa.

```sourcepawn
bool Campaign_GetLocalizedNameFromMapCode(const char[] mapCode, int client, char[] buffer, int maxlength, Localizer localizer = null)
```

**Parámetros:**
- `mapCode` - Código de mapa completo ("L4D2C1", "c5m1_hotel")
- `client` - Índice del cliente
- `buffer` - Buffer de salida
- `maxlength` - Tamaño del buffer
- `localizer` - Instancia de Localizer

**Retorna:** `true` si se encontró traducción, `false` si usa fallback

**Ejemplos:**
```sourcepawn
char campaignName[64];

// Desde código de mapa completo
Campaign_GetLocalizedNameFromMapCode("c1m1_hotel", client, campaignName, sizeof(campaignName), g_Localizer);
// Extrae "c1" → busca traducción → "Dead Center"

// Con prefijo L4D2
Campaign_GetLocalizedNameFromMapCode("L4D2C5M4_quarter", client, campaignName, sizeof(campaignName), g_Localizer);
// Limpia prefijo → extrae "c5" → "The Parish"
```

### Funciones de Localización de Capítulos

#### `Chapter_GetLocalizedName()`
Obtiene el nombre localizado de un capítulo con sensibilidad al modo de juego.

```sourcepawn
bool Chapter_GetLocalizedName(const char[] mapCode, int client, char[] buffer, int maxlength, Localizer localizer = null)
```

**Parámetros:**
- `mapCode` - Código completo del mapa ("c1m1_hotel", "c5m4_quarter")
- `client` - Índice del cliente
- `buffer` - Buffer de salida para nombre del capítulo
- `maxlength` - Tamaño del buffer
- `localizer` - Instancia de Localizer

**Retorna:** `true` si se encontró traducción, `false` si usa fallback

**Formato de Clave Generada:**
```
"#L4D360UI_LevelName_{GAMEMODE}_{mapcode}"
```

**Ejemplos:**
```sourcepawn
char chapterName[128];

// En modo Cooperativo
Chapter_GetLocalizedName("c1m1_hotel", client, chapterName, sizeof(chapterName), g_Localizer);
// Genera: "#L4D360UI_LevelName_COOP_c1m1_hotel" 
// Resultado: "The Hotel"

// En modo Versus  
Chapter_GetLocalizedName("c5m4_quarter", client, chapterName, sizeof(chapterName), g_Localizer);
// Genera: "#L4D360UI_LevelName_VERSUS_c5m4_quarter"
// Resultado: "The Quarter"

// Capítulo personalizado
Chapter_GetLocalizedName("c14m1_lighthouse", client, chapterName, sizeof(chapterName), g_Localizer);
// Sin traducción → Fallback: "c14m1_lighthouse"
```

#### `Chapter_GetFullLocalizedName()`
Obtiene tanto el nombre de campaña como capítulo en una sola llamada.

```sourcepawn
bool Chapter_GetFullLocalizedName(const char[] mapCode, int client, 
                                 char[] campaignOut, int campaignLen,
                                 char[] chapterOut, int chapterLen,
                                 Localizer localizer = null)
```

**Parámetros:**
- `mapCode` - Código completo del mapa
- `client` - Índice del cliente  
- `campaignOut` - Buffer para nombre de campaña
- `campaignLen` - Tamaño del buffer de campaña
- `chapterOut` - Buffer para nombre de capítulo
- `chapterLen` - Tamaño del buffer de capítulo  
- `localizer` - Instancia de Localizer

**Retorna:** `true` si ambos nombres se obtuvieron exitosamente

**Ejemplos:**
```sourcepawn
char campaignName[64], chapterName[128];

if (Chapter_GetFullLocalizedName("c1m1_hotel", client, 
                                campaignName, sizeof(campaignName),
                                chapterName, sizeof(chapterName), 
                                g_Localizer))
{
    PrintToChat(client, "%s - %s", campaignName, chapterName);
    // Resultado: "Dead Center - The Hotel"
}

// Uso con GetCurrentMap()
char currentMap[64];
GetCurrentMap(currentMap, sizeof(currentMap));

Chapter_GetFullLocalizedName(currentMap, client,
                            campaignName, sizeof(campaignName),
                            chapterName, sizeof(chapterName),
                            g_Localizer);
                            
PrintHintText(client, "Jugando: %s\nCapítulo: %s", campaignName, chapterName);
```

### Funciones de Utilidad

#### `Campaign_GetGameModeString()`
Convierte el tipo de modo de juego a string para claves de traducción.

```sourcepawn
char[] Campaign_GetGameModeString(int modeType)
```

**Parámetros:**
- `modeType` - Tipo de modo devuelto por `L4D_GetGameModeType()`

**Retorna:** String del modo de juego

**Mapeo de Modos:**
```sourcepawn
GAMEMODE_COOP     → "COOP"
GAMEMODE_VERSUS   → "VERSUS"  
GAMEMODE_SURVIVAL → "SURVIVAL"
GAMEMODE_SCAVENGE → "SCAVENGE"
default           → "COOP"
```

**Ejemplos:**
```sourcepawn
int currentMode = L4D_GetGameModeType();
char modeString[16];
strcopy(modeString, sizeof(modeString), Campaign_GetGameModeString(currentMode));

PrintToServer("Modo actual: %s", modeString);

// Generar clave específica del modo
char translationKey[64];
Format(translationKey, sizeof(translationKey), "#L4D360UI_LevelName_%s_c1m1_hotel", modeString);
// Resultado: "#L4D360UI_LevelName_COOP_c1m1_hotel"
```

#### `StrUpper()`
Convierte una cadena a mayúsculas (modificación in-place).

```sourcepawn
void StrUpper(char[] str)
```

**Parámetros:**
- `str` - Cadena a convertir (se modifica directamente)

**Ejemplos:**
```sourcepawn
char code[16];
strcopy(code, sizeof(code), "c5m4");
StrUpper(code);
PrintToServer("Mayúsculas: %s", code); // "C5M4"

// Uso en generación de claves
char campaignCode[8] = "c1";
StrUpper(campaignCode);
Format(translationKey, sizeof(translationKey), "#L4D360UI_CampaignName_%s", campaignCode);
// Resultado: "#L4D360UI_CampaignName_C1"
```

### Funciones de Debug

#### `Campaign_DebugMapCode()`
Imprime información completa de debug sobre el procesamiento de un código de mapa.

```sourcepawn
void Campaign_DebugMapCode(const char[] mapCode, int client = 0, Localizer localizer = null)
```

**Parámetros:**
- `mapCode` - Código de mapa a analizar
- `client` - Cliente para probar detección de idioma (opcional)
- `localizer` - Instancia de Localizer para probar traducciones (opcional)

**Información Mostrada:**
- Código original y limpio
- Código de campaña extraído
- Clave de traducción generada
- Clave de traducción de capítulo
- Resultados de traducción con cliente real

**Ejemplos:**
```sourcepawn
// Debug básico
Campaign_DebugMapCode("L4D2C5M4_quarter");

/* Salida:
[Campaign_Debug] Analyzing map code: L4D2C5M4_quarter
[Campaign_Debug] Clean code: C5M4_quarter (prefix removed: yes)
[Campaign_Debug] Campaign code: c5 (extracted: yes)
[Campaign_Debug] Translation key: #L4D360UI_CampaignName_C5
[Campaign_Debug] Chapter translation key: #L4D360UI_LevelName_COOP_C5M4_quarter
*/

// Debug completo con cliente
Campaign_DebugMapCode("c1m1_hotel", client, g_Localizer);

/* Salida adicional:
[Campaign_Debug] Testing with client Player (language: spanish)
[Campaign_Debug] Localized result (success: yes):
[Campaign_Debug] - Campaign: Centro Urbano
[Campaign_Debug] - Chapter: El Hotel
*/
```

## 📝 Ejemplos Prácticos

### Sistema de Información de Mapa Actual

```sourcepawn
public Action CMD_MapInfo(int client, int args)
{
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    char campaignName[64], chapterName[128];
    
    if (Chapter_GetFullLocalizedName(currentMap, client,
                                    campaignName, sizeof(campaignName),
                                    chapterName, sizeof(chapterName),
                                    g_Localizer))
    {
        PrintToChat(client, "\x04Mapa Actual:");
        PrintToChat(client, "\x03Campaña: \x01%s", campaignName);
        PrintToChat(client, "\x03Capítulo: \x01%s", chapterName);
        
        // Información adicional del modo
        int gameMode = L4D_GetGameModeType();
        char modeString[16];
        strcopy(modeString, sizeof(modeString), Campaign_GetGameModeString(gameMode));
        PrintToChat(client, "\x03Modo: \x01%s", modeString);
    }
    else
    {
        PrintToChat(client, "\x04No se pudo obtener información del mapa: %s", currentMap);
    }
    
    return Plugin_Handled;
}
```

### Hook de Cambio de Mapa con Anuncios

```sourcepawn
public void OnMapStart()
{
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    // Delay para asegurar que los clientes estén conectados
    CreateTimer(5.0, Timer_AnnounceMap, 0, TIMER_FLAG_NO_MAPCHANGE);
}

public Action Timer_AnnounceMap(Handle timer)
{
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    for (int client = 1; client <= MaxClients; client++)
    {
        if (IsClientInGame(client) && !IsFakeClient(client))
        {
            char campaignName[64], chapterName[128];
            
            if (Chapter_GetFullLocalizedName(currentMap, client,
                                            campaignName, sizeof(campaignName),
                                            chapterName, sizeof(chapterName),
                                            g_Localizer))
            {
                PrintHintText(client, "Bienvenido a:\n%s\n%s", campaignName, chapterName);
            }
        }
    }
    
    return Plugin_Stop;
}
```

### Sistema de Votación de Mapas con Nombres Localizados

```sourcepawn
#define MAX_MAPS 20

char g_MapList[MAX_MAPS][64];
int g_MapCount = 0;

public void LoadMapList()
{
    // Cargar lista de mapas desde archivo
    g_MapCount = 0;
    strcopy(g_MapList[g_MapCount++], sizeof(g_MapList[]), "c1m1_hotel");
    strcopy(g_MapList[g_MapCount++], sizeof(g_MapList[]), "c2m1_highway");
    strcopy(g_MapList[g_MapCount++], sizeof(g_MapList[]), "c5m1_waterfront");
    // ... más mapas
}

public Action CMD_VoteMap(int client, int args)
{
    Menu menu = new Menu(Menu_MapVote);
    menu.SetTitle("Votar Mapa:");
    
    for (int i = 0; i < g_MapCount; i++)
    {
        char campaignName[64], chapterName[128];
        char displayName[192];
        
        if (Chapter_GetFullLocalizedName(g_MapList[i], client,
                                        campaignName, sizeof(campaignName),
                                        chapterName, sizeof(chapterName),
                                        g_Localizer))
        {
            Format(displayName, sizeof(displayName), "%s - %s", campaignName, chapterName);
        }
        else
        {
            strcopy(displayName, sizeof(displayName), g_MapList[i]);
        }
        
        menu.AddItem(g_MapList[i], displayName);
    }
    
    menu.Display(client, MENU_TIME_FOREVER);
    return Plugin_Handled;
}

public int Menu_MapVote(Menu menu, MenuAction action, int param1, int param2)
{
    if (action == MenuAction_Select)
    {
        char mapCode[64];
        menu.GetItem(param2, mapCode, sizeof(mapCode));
        
        // Procesar voto del mapa
        ProcessMapVote(param1, mapCode);
    }
    else if (action == MenuAction_End)
    {
        delete menu;
    }
    
    return 0;
}
```

### Sistema de Progreso de Campaña

```sourcepawn
public void OnRoundEnd()
{
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    char mapCode[16];
    if (Campaign_ExtractMapCode(currentMap, mapCode, sizeof(mapCode)))
    {
        // Extraer número de capítulo
        int chapterNum = GetChapterNumber(mapCode);
        int totalChapters = GetTotalChaptersForCampaign(mapCode);
        
        if (chapterNum > 0 && totalChapters > 0)
        {
            char campaignName[64];
            Campaign_GetLocalizedNameFromMapCode(currentMap, 0, campaignName, sizeof(campaignName), g_Localizer);
            
            for (int client = 1; client <= MaxClients; client++)
            {
                if (IsClientInGame(client) && !IsFakeClient(client))
                {
                    PrintToChat(client, "\x04Progreso de Campaña:");
                    PrintToChat(client, "\x03%s: \x01Capítulo %d/%d completado", 
                               campaignName, chapterNum, totalChapters);
                    
                    // Mostrar barra de progreso
                    float progress = float(chapterNum) / float(totalChapters);
                    ShowProgressBar(client, progress);
                }
            }
        }
    }
}

int GetChapterNumber(const char[] mapCode)
{
    // Extraer número después de 'm'
    int pos = FindCharInString(mapCode, 'm');
    if (pos != -1)
    {
        return StringToInt(mapCode[pos + 1]);
    }
    return 0;
}
```

### Sistema de Estadísticas por Campaña

```sourcepawn
StringMap g_CampaignStats;

public void OnPluginStart()
{
    g_CampaignStats = new StringMap();
}

public void OnMapStart()
{
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    char campaignCode[8];
    if (Campaign_ExtractCampaignCode(currentMap, campaignCode, sizeof(campaignCode)))
    {
        int playCount = 0;
        g_CampaignStats.GetValue(campaignCode, playCount);
        g_CampaignStats.SetValue(campaignCode, playCount + 1);
    }
}

public Action CMD_CampaignStats(int client, int args)
{
    PrintToChat(client, "\x04=== Estadísticas de Campañas ===");
    
    StringMapSnapshot snapshot = g_CampaignStats.Snapshot();
    
    for (int i = 0; i < snapshot.Length; i++)
    {
        char campaignCode[8];
        snapshot.GetKey(i, campaignCode, sizeof(campaignCode));
        
        int playCount;
        g_CampaignStats.GetValue(campaignCode, playCount);
        
        char campaignName[64];
        if (Campaign_GetLocalizedName(campaignCode, client, campaignName, sizeof(campaignName), g_Localizer))
        {
            PrintToChat(client, "\x03%s: \x01%d veces jugada", campaignName, playCount);
        }
    }
    
    delete snapshot;
    return Plugin_Handled;
}
```

## 🔍 Debugging y Troubleshooting

### Problemas Comunes

#### 1. Códigos de mapa no reconocidos
```sourcepawn
public void ValidateMapCode(const char[] mapCode)
{
    char campaignCode[8];
    if (!Campaign_ExtractCampaignCode(mapCode, campaignCode, sizeof(campaignCode)))
    {
        LogError("Código de mapa inválido: %s", mapCode);
        LogError("Formato esperado: c[número]m[número]_nombre");
        return;
    }
    
    char translationKey[64];
    if (!Campaign_GenerateTranslationKey(campaignCode, translationKey, sizeof(translationKey)))
    {
        LogError("No se pudo generar clave de traducción para: %s", campaignCode);
        return;
    }
    
    LogMessage("Código válido: %s → Clave: %s", campaignCode, translationKey);
}
```

#### 2. Traducciones no encontradas
```sourcepawn
public void DiagnoseTranslation(int client, const char[] mapCode)
{
    char campaignCode[8];
    Campaign_ExtractCampaignCode(mapCode, campaignCode, sizeof(campaignCode));
    
    char translationKey[64];
    Campaign_GenerateTranslationKey(campaignCode, translationKey, sizeof(translationKey));
    
    char result[64];
    if (Lang_GetValveTranslation(client, translationKey, result, sizeof(result), g_Localizer))
    {
        LogMessage("✅ Traducción encontrada: %s → %s", translationKey, result);
    }
    else
    {
        LogWarning("❌ Traducción no encontrada: %s", translationKey);
        LogWarning("Posibles causas:");
        LogWarning("  - Localizer no está listo");
        LogWarning("  - Clave no existe en archivos de traducción");
        LogWarning("  - Idioma del cliente no soportado");
    }
}
```

#### 3. Modo de juego incorrecto
```sourcepawn
public void DiagnoseGameMode()
{
    int gameMode = L4D_GetGameModeType();
    char modeString[16];
    strcopy(modeString, sizeof(modeString), Campaign_GetGameModeString(gameMode));
    
    LogMessage("Modo de juego actual: %d → %s", gameMode, modeString);
    
    // Verificar si el modo es válido
    switch (gameMode)
    {
        case GAMEMODE_COOP, GAMEMODE_VERSUS, GAMEMODE_SURVIVAL, GAMEMODE_SCAVENGE:
            LogMessage("✅ Modo válido");
        default:
            LogWarning("❌ Modo desconocido, usando COOP como fallback");
    }
}
```

### Herramientas de Debug Avanzadas

#### Debug Completo de Pipeline
```sourcepawn
public Action CMD_DebugPipeline(int client, int args)
{
    if (args < 1)
    {
        ReplyToCommand(client, "Uso: sm_debug_pipeline <código_mapa>");
        return Plugin_Handled;
    }
    
    char mapCode[64];
    GetCmdArg(1, mapCode, sizeof(mapCode));
    
    PrintToConsole(client, "=== PIPELINE DEBUG ===");
    PrintToConsole(client, "Input: %s", mapCode);
    
    // Paso 1: Limpiar prefijo
    char cleanCode[32];
    bool prefixRemoved = Campaign_RemoveMapPrefix(mapCode, cleanCode, sizeof(cleanCode));
    PrintToConsole(client, "1. Clean: %s (prefijo eliminado: %s)", cleanCode, prefixRemoved ? "sí" : "no");
    
    // Paso 2: Extraer código de campaña
    char campaignCode[8];
    bool campaignExtracted = Campaign_ExtractCampaignCode(mapCode, campaignCode, sizeof(campaignCode));
    PrintToConsole(client, "2. Campaign: %s (extraído: %s)", campaignCode, campaignExtracted ? "sí" : "no");
    
    // Paso 3: Extraer código de mapa completo
    char fullMapCode[16];
    bool mapExtracted = Campaign_ExtractMapCode(mapCode, fullMapCode, sizeof(fullMapCode));
    PrintToConsole(client, "3. Map Code: %s (extraído: %s)", fullMapCode, mapExtracted ? "sí" : "no");
    
    // Paso 4: Generar claves
    if (campaignExtracted)
    {
        char campaignKey[64];
        Campaign_GenerateTranslationKey(campaignCode, campaignKey, sizeof(campaignKey));
        PrintToConsole(client, "4. Campaign Key: %s", campaignKey);
        
        int gameMode = L4D_GetGameModeType();
        char modeString[16];
        strcopy(modeString, sizeof(modeString), Campaign_GetGameModeString(gameMode));
        
        char chapterKey[64];
        Format(chapterKey, sizeof(chapterKey), "#L4D360UI_LevelName_%s_%s", modeString, cleanCode);
        PrintToConsole(client, "5. Chapter Key: %s", chapterKey);
    }
    
    // Usar la función de debug integrada
    Campaign_DebugMapCode(mapCode, client, g_Localizer);
    
    return Plugin_Handled;
}
```

## 📊 Performance y Optimización

### Mejores Prácticas

1. **Cache códigos de mapa frecuentemente utilizados:**
```sourcepawn
StringMap g_MapCodeCache;

public void OnPluginStart()
{
    g_MapCodeCache = new StringMap();
}

bool GetCachedCampaignCode(const char[] mapCode, char[] campaignCode, int maxlength)
{
    if (g_MapCodeCache.GetString(mapCode, campaignCode, maxlength))
    {
        return true;
    }
    
    if (Campaign_ExtractCampaignCode(mapCode, campaignCode, maxlength))
    {
        g_MapCodeCache.SetString(mapCode, campaignCode);
        return true;
    }
    
    return false;
}
```

2. **Evitar procesamiento repetitivo:**
```sourcepawn
// ❌ Ineficiente
public void OnGameFrame()
{
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    Campaign_ExtractCampaignCode(currentMap, ...); // Se ejecuta cada frame
}

// ✅ Eficiente
char g_CurrentCampaignCode[8];

public void OnMapStart()
{
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    Campaign_ExtractCampaignCode(currentMap, g_CurrentCampaignCode, sizeof(g_CurrentCampaignCode));
}
```

3. **Usar debug solo en desarrollo:**
```sourcepawn
#if defined DEBUG
    Campaign_DebugMapCode(mapCode, client, g_Localizer);
#endif
```

## 🌍 Mapeo de Campañas Oficiales

### Campañas L4D2 Originales
| Código | Nombre Inglés | Clave de Traducción |
|--------|---------------|-------------------|
| `c1` | Dead Center | `#L4D360UI_CampaignName_C1` |
| `c2` | Dark Carnival | `#L4D360UI_CampaignName_C2` |
| `c3` | Swamp Fever | `#L4D360UI_CampaignName_C3` |
| `c4` | Hard Rain | `#L4D360UI_CampaignName_C4` |
| `c5` | The Parish | `#L4D360UI_CampaignName_C5` |

### Campañas DLC
| Código | Nombre Inglés | Clave de Traducción |
|--------|---------------|-------------------|
| `c6` | The Passing | `#L4D360UI_CampaignName_C6` |
| `c7` | The Sacrifice | `#L4D360UI_CampaignName_C7` |
| `c13` | Cold Stream | `#L4D360UI_CampaignName_C13` |
| `c14` | The Last Stand | `#L4D360UI_CampaignName_C14` |

### Campañas L4D1 Portadas
| Código | Nombre Inglés | Clave de Traducción |
|--------|---------------|-------------------|
| `c8` | No Mercy | `#L4D360UI_CampaignName_C8` |
| `c9` | Crash Course | `#L4D360UI_CampaignName_C9` |
| `c10` | Death Toll | `#L4D360UI_CampaignName_C10` |
| `c11` | Dead Air | `#L4D360UI_CampaignName_C11` |
| `c12` | Blood Harvest | `#L4D360UI_CampaignName_C12` |

---

**Siguiente:** [Ejemplos de Integración](examples.md)

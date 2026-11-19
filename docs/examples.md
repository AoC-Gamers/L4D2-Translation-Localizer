# Ejemplos de Integración

Ejemplos prácticos de uso conjunto de `language_manager.inc` y `campaign_manager.inc` en plugins reales.

> **NOTA IMPORTANTE**: Estos ejemplos usan SM-Localizer para acceder a traducciones Valve intrínsecas del juego.
> El constructor `new Localizer()` sin parámetros inicializa el sistema para usar traducciones oficiales de Valve.

## Plugin Base Template

### Estructura Básica
```sourcepawn
#pragma semicolon 1
#pragma newdecls required

#include <sourcemod>
#include <language_manager>
#include <campaign_manager>
#include <localizer>
#include <left4dhooks>

public Plugin myinfo = 
{
    name = "Mi Plugin con Localización",
    author = "Tu Nombre",
    description = "Plugin ejemplo con sistema de localización",
    version = "1.0.0",
    url = "https://github.com/tu-usuario/tu-plugin"
};

Localizer g_Localizer;

public void OnPluginStart()
{
    // Inicializar Localizer
    g_Localizer = new Localizer();
    g_Localizer.Delegate_InitCompleted(OnPhrasesReady);
    
    // Registrar comandos
    RegConsoleCmd("sm_mapinfo", CMD_MapInfo, "Mostrar información del mapa actual");
    RegConsoleCmd("sm_language", CMD_Language, "Mostrar información de idioma");
}

public void OnPhrasesReady()
{
    // Localizer está listo para usar traducciones Valve
    LogMessage("Sistema de localización inicializado correctamente");
}

public void OnAllPluginsLoaded()
{
    // Verificar que Localizer esté listo
    if (!g_Localizer.IsReady())
    {
        LogError("Localizer no está listo - algunas traducciones pueden fallar");
        // No usar SetFailState aquí, el sistema puede tardar en inicializarse
    }
}
```

## Ejemplo 1: Sistema de Bienvenida Avanzado

### Plugin Completo: Welcome System
```sourcepawn
public void OnClientPostAdminCheck(int client)
{
    if (IsFakeClient(client))
        return;
        
    // Delay para asegurar que el cliente esté completamente conectado
    CreateTimer(2.0, Timer_WelcomeClient, GetClientUserId(client), TIMER_FLAG_NO_MAPCHANGE);
}

public Action Timer_WelcomeClient(Handle timer, int userid)
{
    int client = GetClientOfUserId(userid);
    if (!client || !IsClientInGame(client))
        return Plugin_Stop;
    
    // Obtener información del cliente
    char clientName[MAX_NAME_LENGTH];
    char clientLang[32];
    GetClientName(client, clientName, sizeof(clientName));
    Lang_GetSafeClientLanguage(client, clientLang, sizeof(clientLang));
    
    // Obtener información del mapa actual
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    char campaignName[64], chapterName[128];
    bool hasMapInfo = Chapter_GetFullLocalizedName(currentMap, client,
                                                  campaignName, sizeof(campaignName),
                                                  chapterName, sizeof(chapterName),
                                                  g_Localizer);
    
    // Mensaje de bienvenida personalizado por idioma
    LanguageID langId = Lang_GetClientLanguage(client);
    
    if (Lang_IsSpanishVariant(langId))
    {
        PrintToChat(client, "\x04¡Bienvenido \x03%s\x04!", clientName);
        PrintToChat(client, "\x01Tu idioma: \x05%s", clientLang);
        
        if (hasMapInfo)
        {
            PrintToChat(client, "\x01Estás jugando: \x03%s \x01- \x05%s", campaignName, chapterName);
        }
        
        PrintToChat(client, "\x01Escribe \x04!ayuda \x01para ver los comandos disponibles");
    }
    else if (Lang_IsEnglish(langId))
    {
        PrintToChat(client, "\x04Welcome \x03%s\x04!", clientName);
        PrintToChat(client, "\x01Your language: \x05%s", clientLang);
        
        if (hasMapInfo)
        {
            PrintToChat(client, "\x01You're playing: \x03%s \x01- \x05%s", campaignName, chapterName);
        }
        
        PrintToChat(client, "\x01Type \x04!help \x01to see available commands");
    }
    else if (Lang_IsRomanceLanguage(langId))
    {
        // Mensaje genérico para lenguas romances
        PrintToChat(client, "\x04Bienvenue/Benvenuto \x03%s\x04!", clientName);
        PrintToChat(client, "\x01Language/Langue/Lingua: \x05%s", clientLang);
        
        if (hasMapInfo)
        {
            PrintToChat(client, "\x01Map/Mappa/Carte: \x03%s \x01- \x05%s", campaignName, chapterName);
        }
    }
    else
    {
        // Fallback en inglés
        PrintToChat(client, "\x04Welcome \x03%s\x04!", clientName);
        PrintToChat(client, "\x01Language: \x05%s", clientLang);
        
        if (hasMapInfo)
        {
            PrintToChat(client, "\x01Current map: \x03%s \x01- \x05%s", campaignName, chapterName);
        }
    }
    
    return Plugin_Stop;
}
```

## Ejemplo 2: Sistema de Votación de Mapas

### Plugin: Advanced Map Voting
```sourcepawn
#define MAX_VOTE_MAPS 15

char g_MapPool[MAX_VOTE_MAPS][64];
int g_MapPoolSize = 0;
StringMap g_MapVotes;
bool g_VoteInProgress = false;

public void OnPluginStart()
{
    // Inicializar Localizer
    g_Localizer = new Localizer();
    g_Localizer.Delegate_InitCompleted(OnPhrasesReady);
    
    g_MapVotes = new StringMap();
    LoadMapPool();
    
    RegAdminCmd("sm_votemapa", CMD_VoteMap, ADMFLAG_CHANGEMAP, "Iniciar votación de mapa");
    RegConsoleCmd("sm_mapas", CMD_ListMaps, "Ver mapas disponibles");
}

public void OnPhrasesReady()
{
    // Localizer está listo para usar traducciones Valve
    LogMessage("Sistema de localización inicializado correctamente");
}

void LoadMapPool()
{
    // Cargar pool de mapas (en producción, desde archivo)
    g_MapPoolSize = 0;
    
    // Campañas L4D2 originales
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c1m1_hotel");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c2m1_highway");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c3m1_plankcountry");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c4m1_milltown_a");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c5m1_waterfront");
    
    // Campañas DLC
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c6m1_riverbank");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c7m01_docks");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c14m1_junkyard");
    
    // Campañas L4D1
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c8m1_apartment");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c10m1_caves");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c11m1_greenhouse");
    strcopy(g_MapPool[g_MapPoolSize++], sizeof(g_MapPool[]), "c12m1_hilltop");
}

public Action CMD_ListMaps(int client, int args)
{
    if (!client)
        return Plugin_Handled;
    
    char clientLang[32];
    Lang_GetSafeClientLanguage(client, clientLang, sizeof(clientLang));
    LanguageID langId = Lang_GetClientLanguage(client);
    
    // Título localizado
    if (Lang_IsSpanishVariant(langId))
    {
        PrintToChat(client, "\x04=== Mapas Disponibles ===");
    }
    else
    {
        PrintToChat(client, "\x04=== Available Maps ===");
    }
    
    for (int i = 0; i < g_MapPoolSize; i++)
    {
        char campaignName[64], chapterName[128];
        char displayName[192];
        
        if (Chapter_GetFullLocalizedName(g_MapPool[i], client,
                                        campaignName, sizeof(campaignName),
                                        chapterName, sizeof(chapterName),
                                        g_Localizer))
        {
            Format(displayName, sizeof(displayName), "%s - %s", campaignName, chapterName);
        }
        else
        {
            strcopy(displayName, sizeof(displayName), g_MapPool[i]);
        }
        
        PrintToChat(client, "\x03%d. \x01%s", i + 1, displayName);
    }
    
    return Plugin_Handled;
}

public Action CMD_VoteMap(int client, int args)
{
    if (g_VoteInProgress)
    {
        ReplyToCommand(client, "Ya hay una votación en progreso");
        return Plugin_Handled;
    }
    
    StartMapVote();
    return Plugin_Handled;
}

void StartMapVote()
{
    g_VoteInProgress = true;
    g_MapVotes.Clear();
    
    Menu voteMenu = new Menu(Menu_VoteHandler);
    
    // Título dinámico según el primer cliente conectado
    int firstClient = GetFirstValidClient();
    if (firstClient > 0)
    {
        if (Lang_IsSpanishVariant(Lang_GetClientLanguage(firstClient)))
        {
            voteMenu.SetTitle("Votar por el próximo mapa:");
        }
        else
        {
            voteMenu.SetTitle("Vote for next map:");
        }
    }
    else
    {
        voteMenu.SetTitle("Vote for next map:");
    }
    
    // Seleccionar mapas aleatorios para la votación
    ArrayList randomMaps = new ArrayList();
    for (int i = 0; i < g_MapPoolSize; i++)
    {
        randomMaps.Push(i);
    }
    randomMaps.Sort(Sort_Random, Sort_Integer);
    
    int maxOptions = (g_MapPoolSize < 6) ? g_MapPoolSize : 6;
    
    for (int i = 0; i < maxOptions; i++)
    {
        int mapIndex = randomMaps.Get(i);
        char mapCode[64];
        strcopy(mapCode, sizeof(mapCode), g_MapPool[mapIndex]);
        
        // Generar nombre de display para cada cliente será manejado en el menú
        voteMenu.AddItem(mapCode, mapCode);
    }
    
    delete randomMaps;
    
    // Mostrar menú a todos los clientes
    for (int client = 1; client <= MaxClients; client++)
    {
        if (IsClientInGame(client) && !IsFakeClient(client))
        {
            // Personalizar menú por cliente
            Menu clientMenu = voteMenu.Clone();
            CustomizeMenuForClient(clientMenu, client);
            clientMenu.Display(client, 30);
        }
    }
    
    delete voteMenu;
    
    // Timer para finalizar votación
    CreateTimer(30.0, Timer_EndVote, 0, TIMER_FLAG_NO_MAPCHANGE);
    
    // Anunciar votación
    for (int client = 1; client <= MaxClients; client++)
    {
        if (IsClientInGame(client) && !IsFakeClient(client))
        {
            if (Lang_IsSpanishVariant(Lang_GetClientLanguage(client)))
            {
                PrintToChat(client, "\x04¡Votación de mapa iniciada! Tienes 30 segundos para votar.");
            }
            else
            {
                PrintToChat(client, "\x04Map vote started! You have 30 seconds to vote.");
            }
        }
    }
}

void CustomizeMenuForClient(Menu menu, int client)
{
    // Personalizar elementos del menú con nombres localizados
    int itemCount = menu.ItemCount;
    
    for (int i = 0; i < itemCount; i++)
    {
        char mapCode[64], display[192];
        menu.GetItem(i, mapCode, sizeof(mapCode), _, display, sizeof(display));
        
        char campaignName[64], chapterName[128];
        if (Chapter_GetFullLocalizedName(mapCode, client,
                                        campaignName, sizeof(campaignName),
                                        chapterName, sizeof(chapterName),
                                        g_Localizer))
        {
            Format(display, sizeof(display), "%s - %s", campaignName, chapterName);
            menu.UpdateItem(i, mapCode, display);
        }
    }
}

public int Menu_VoteHandler(Menu menu, MenuAction action, int param1, int param2)
{
    switch (action)
    {
        case MenuAction_Select:
        {
            char mapCode[64];
            menu.GetItem(param2, mapCode, sizeof(mapCode));
            
            // Registrar voto
            int currentVotes = 0;
            g_MapVotes.GetValue(mapCode, currentVotes);
            g_MapVotes.SetValue(mapCode, currentVotes + 1);
            
            // Confirmar voto al cliente
            char campaignName[64], chapterName[128];
            if (Chapter_GetFullLocalizedName(mapCode, param1,
                                            campaignName, sizeof(campaignName),
                                            chapterName, sizeof(chapterName),
                                            g_Localizer))
            {
                if (Lang_IsSpanishVariant(Lang_GetClientLanguage(param1)))
                {
                    PrintToChat(param1, "\x04Has votado por: \x03%s - %s", campaignName, chapterName);
                }
                else
                {
                    PrintToChat(param1, "\x04You voted for: \x03%s - %s", campaignName, chapterName);
                }
            }
        }
        case MenuAction_End:
        {
            delete menu;
        }
    }
    
    return 0;
}

public Action Timer_EndVote(Handle timer)
{
    if (!g_VoteInProgress)
        return Plugin_Stop;
    
    // Encontrar el mapa ganador
    char winnerMap[64];
    int maxVotes = 0;
    
    StringMapSnapshot snapshot = g_MapVotes.Snapshot();
    
    for (int i = 0; i < snapshot.Length; i++)
    {
        char mapCode[64];
        int votes;
        
        snapshot.GetKey(i, mapCode, sizeof(mapCode));
        g_MapVotes.GetValue(mapCode, votes);
        
        if (votes > maxVotes)
        {
            maxVotes = votes;
            strcopy(winnerMap, sizeof(winnerMap), mapCode);
        }
    }
    
    delete snapshot;
    
    g_VoteInProgress = false;
    
    if (maxVotes > 0)
    {
        // Anunciar ganador
        for (int client = 1; client <= MaxClients; client++)
        {
            if (IsClientInGame(client) && !IsFakeClient(client))
            {
                char campaignName[64], chapterName[128];
                if (Chapter_GetFullLocalizedName(winnerMap, client,
                                                campaignName, sizeof(campaignName),
                                                chapterName, sizeof(chapterName),
                                                g_Localizer))
                {
                    if (Lang_IsSpanishVariant(Lang_GetClientLanguage(client)))
                    {
                        PrintToChat(client, "\x04¡Mapa ganador: \x03%s - %s \x04(%d votos)!", 
                                   campaignName, chapterName, maxVotes);
                        PrintToChat(client, "\x01Cambiando mapa en 10 segundos...");
                    }
                    else
                    {
                        PrintToChat(client, "\x04Winning map: \x03%s - %s \x04(%d votes)!", 
                                   campaignName, chapterName, maxVotes);
                        PrintToChat(client, "\x01Changing map in 10 seconds...");
                    }
                }
            }
        }
        
        // Cambiar mapa
        CreateTimer(10.0, Timer_ChangeMap, 0, TIMER_FLAG_NO_MAPCHANGE);
        strcopy(g_NextMap, sizeof(g_NextMap), winnerMap);
    }
    else
    {
        PrintToChatAll("\x04No se recibieron votos. Mapa no cambiado.");
    }
    
    return Plugin_Stop;
}

char g_NextMap[64];

public Action Timer_ChangeMap(Handle timer)
{
    ServerCommand("changelevel %s", g_NextMap);
    return Plugin_Stop;
}

int GetFirstValidClient()
{
    for (int client = 1; client <= MaxClients; client++)
    {
        if (IsClientInGame(client) && !IsFakeClient(client))
            return client;
    }
    return 0;
}
```

## Ejemplo 3: Sistema de Estadísticas Avanzado

### Plugin: Player Statistics with Localization
```sourcepawn
enum struct PlayerStats
{
    int kills;
    int deaths;
    int revives;
    int heals;
    char preferredLanguage[32];
    char mostPlayedCampaign[8];
    int totalPlayTime;
}

PlayerStats g_PlayerStats[MAXPLAYERS + 1];
StringMap g_CampaignPlayTime;
StringMap g_LanguageStats;

public void OnPluginStart()
{
    // Inicializar Localizer
    g_Localizer = new Localizer();
    g_Localizer.Delegate_InitCompleted(OnPhrasesReady);
    
    g_CampaignPlayTime = new StringMap();
    g_LanguageStats = new StringMap();
    
    RegConsoleCmd("sm_stats", CMD_PlayerStats, "Ver estadísticas personales");
    RegConsoleCmd("sm_serverstats", CMD_ServerStats, "Ver estadísticas del servidor");
    
    // Hooks de eventos
    HookEvent("player_death", Event_PlayerDeath);
    HookEvent("revive_success", Event_ReviveSuccess);
    HookEvent("heal_success", Event_HealSuccess);
}

public void OnPhrasesReady()
{
    // Localizer está listo para usar traducciones Valve
    LogMessage("Sistema de localización inicializado correctamente");
}

public void OnClientPostAdminCheck(int client)
{
    if (IsFakeClient(client))
        return;
    
    // Inicializar estadísticas del cliente
    g_PlayerStats[client].kills = 0;
    g_PlayerStats[client].deaths = 0;
    g_PlayerStats[client].revives = 0;
    g_PlayerStats[client].heals = 0;
    g_PlayerStats[client].totalPlayTime = 0;
    
    // Registrar idioma preferido
    Lang_GetSafeClientLanguage(client, g_PlayerStats[client].preferredLanguage, 
                              sizeof(g_PlayerStats[].preferredLanguage));
    
    // Actualizar estadísticas de idiomas del servidor
    int langCount = 0;
    g_LanguageStats.GetValue(g_PlayerStats[client].preferredLanguage, langCount);
    g_LanguageStats.SetValue(g_PlayerStats[client].preferredLanguage, langCount + 1);
    
    // Determinar campaña más jugada (simulado)
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    char campaignCode[8];
    if (Campaign_ExtractCampaignCode(currentMap, campaignCode, sizeof(campaignCode)))
    {
        strcopy(g_PlayerStats[client].mostPlayedCampaign, 
               sizeof(g_PlayerStats[].mostPlayedCampaign), campaignCode);
    }
}

public void OnClientDisconnect(int client)
{
    if (IsFakeClient(client))
        return;
    
    // Actualizar estadísticas de idiomas
    int langCount = 0;
    if (g_LanguageStats.GetValue(g_PlayerStats[client].preferredLanguage, langCount))
    {
        langCount = (langCount > 0) ? langCount - 1 : 0;
        g_LanguageStats.SetValue(g_PlayerStats[client].preferredLanguage, langCount);
    }
}

public Action Event_PlayerDeath(Event event, const char[] name, bool dontBroadcast)
{
    int victim = GetClientOfUserId(event.GetInt("userid"));
    int attacker = GetClientOfUserId(event.GetInt("attacker"));
    
    if (victim > 0 && victim <= MaxClients)
    {
        g_PlayerStats[victim].deaths++;
    }
    
    if (attacker > 0 && attacker <= MaxClients && attacker != victim)
    {
        g_PlayerStats[attacker].kills++;
    }
    
    return Plugin_Continue;
}

public Action Event_ReviveSuccess(Event event, const char[] name, bool dontBroadcast)
{
    int client = GetClientOfUserId(event.GetInt("userid"));
    if (client > 0 && client <= MaxClients)
    {
        g_PlayerStats[client].revives++;
    }
    
    return Plugin_Continue;
}

public Action Event_HealSuccess(Event event, const char[] name, bool dontBroadcast)
{
    int client = GetClientOfUserId(event.GetInt("userid"));
    if (client > 0 && client <= MaxClients)
    {
        g_PlayerStats[client].heals++;
    }
    
    return Plugin_Continue;
}

public Action CMD_PlayerStats(int client, int args)
{
    if (!client)
        return Plugin_Handled;
    
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    // Encabezado localizado
    if (Lang_IsSpanishVariant(clientLang))
    {
        PrintToChat(client, "\x04=== Tus Estadísticas ===");
    }
    else if (Lang_IsFrench(clientLang))
    {
        PrintToChat(client, "\x04=== Vos Statistiques ===");
    }
    else if (Lang_IsGerman(clientLang))
    {
        PrintToChat(client, "\x04=== Deine Statistiken ===");
    }
    else
    {
        PrintToChat(client, "\x04=== Your Statistics ===");
    }
    
    // Estadísticas básicas
    if (Lang_IsSpanishVariant(clientLang))
    {
        PrintToChat(client, "\x03Eliminaciones: \x01%d", g_PlayerStats[client].kills);
        PrintToChat(client, "\x03Muertes: \x01%d", g_PlayerStats[client].deaths);
        PrintToChat(client, "\x03Reanimaciones: \x01%d", g_PlayerStats[client].revives);
        PrintToChat(client, "\x03Curaciones: \x01%d", g_PlayerStats[client].heals);
    }
    else
    {
        PrintToChat(client, "\x03Kills: \x01%d", g_PlayerStats[client].kills);
        PrintToChat(client, "\x03Deaths: \x01%d", g_PlayerStats[client].deaths);
        PrintToChat(client, "\x03Revives: \x01%d", g_PlayerStats[client].revives);
        PrintToChat(client, "\x03Heals: \x01%d", g_PlayerStats[client].heals);
    }
    
    // Campaña más jugada
    if (strlen(g_PlayerStats[client].mostPlayedCampaign) > 0)
    {
        char campaignName[64];
        if (Campaign_GetLocalizedName(g_PlayerStats[client].mostPlayedCampaign, 
                                     client, campaignName, sizeof(campaignName), g_Localizer))
        {
            if (Lang_IsSpanishVariant(clientLang))
            {
                PrintToChat(client, "\x03Campaña favorita: \x01%s", campaignName);
            }
            else
            {
                PrintToChat(client, "\x03Favorite campaign: \x01%s", campaignName);
            }
        }
    }
    
    // Ratio K/D
    float kdr = (g_PlayerStats[client].deaths > 0) ? 
                float(g_PlayerStats[client].kills) / float(g_PlayerStats[client].deaths) : 
                float(g_PlayerStats[client].kills);
    
    if (Lang_IsSpanishVariant(clientLang))
    {
        PrintToChat(client, "\x03Ratio K/D: \x01%.2f", kdr);
    }
    else
    {
        PrintToChat(client, "\x03K/D Ratio: \x01%.2f", kdr);
    }
    
    return Plugin_Handled;
}

public Action CMD_ServerStats(int client, int args)
{
    if (!client)
        return Plugin_Handled;
    
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    // Encabezado
    if (Lang_IsSpanishVariant(clientLang))
    {
        PrintToChat(client, "\x04=== Estadísticas del Servidor ===");
    }
    else
    {
        PrintToChat(client, "\x04=== Server Statistics ===");
    }
    
    // Estadísticas de idiomas
    if (Lang_IsSpanishVariant(clientLang))
    {
        PrintToChat(client, "\x03Jugadores por idioma:");
    }
    else
    {
        PrintToChat(client, "\x03Players by language:");
    }
    
    StringMapSnapshot langSnapshot = g_LanguageStats.Snapshot();
    
    for (int i = 0; i < langSnapshot.Length; i++)
    {
        char langName[32];
        int count;
        
        langSnapshot.GetKey(i, langName, sizeof(langName));
        g_LanguageStats.GetValue(langName, count);
        
        if (count > 0)
        {
            // Convertir nombre de idioma a display name
            char displayLang[32];
            LanguageID langId = Lang_FindByName(langName);
            
            if (Lang_IsSpanishVariant(clientLang))
            {
                GetSpanishLanguageName(langId, displayLang, sizeof(displayLang));
            }
            else
            {
                GetEnglishLanguageName(langId, displayLang, sizeof(displayLang));
            }
            
            PrintToChat(client, "\x01  %s: \x05%d", displayLang, count);
        }
    }
    
    delete langSnapshot;
    
    // Mapa actual con información
    char currentMap[64];
    GetCurrentMap(currentMap, sizeof(currentMap));
    
    char campaignName[64], chapterName[128];
    if (Chapter_GetFullLocalizedName(currentMap, client,
                                    campaignName, sizeof(campaignName),
                                    chapterName, sizeof(chapterName),
                                    g_Localizer))
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            PrintToChat(client, "\x03Mapa actual: \x01%s - %s", campaignName, chapterName);
        }
        else
        {
            PrintToChat(client, "\x03Current map: \x01%s - %s", campaignName, chapterName);
        }
    }
    
    return Plugin_Handled;
}

void GetSpanishLanguageName(LanguageID langId, char[] buffer, int maxlength)
{
    switch (langId)
    {
        case LANGID_ENGLISH: strcopy(buffer, maxlength, "Inglés");
        case LANGID_SPANISH: strcopy(buffer, maxlength, "Español");
        case LANGID_LATAM: strcopy(buffer, maxlength, "Español (LATAM)");
        case LANGID_FRENCH: strcopy(buffer, maxlength, "Francés");
        case LANGID_GERMAN: strcopy(buffer, maxlength, "Alemán");
        case LANGID_ITALIAN: strcopy(buffer, maxlength, "Italiano");
        case LANGID_PORTUGUESE: strcopy(buffer, maxlength, "Portugués");
        case LANGID_BRAZILIAN: strcopy(buffer, maxlength, "Portugués (Brasil)");
        case LANGID_RUSSIAN: strcopy(buffer, maxlength, "Ruso");
        default: 
        {
            char langName[32];
            Lang_GetLanguageName(langId, langName, sizeof(langName));
            strcopy(buffer, maxlength, langName);
        }
    }
}

void GetEnglishLanguageName(LanguageID langId, char[] buffer, int maxlength)
{
    switch (langId)
    {
        case LANGID_ENGLISH: strcopy(buffer, maxlength, "English");
        case LANGID_SPANISH: strcopy(buffer, maxlength, "Spanish");
        case LANGID_LATAM: strcopy(buffer, maxlength, "Spanish (LATAM)");
        case LANGID_FRENCH: strcopy(buffer, maxlength, "French");
        case LANGID_GERMAN: strcopy(buffer, maxlength, "German");
        case LANGID_ITALIAN: strcopy(buffer, maxlength, "Italian");
        case LANGID_PORTUGUESE: strcopy(buffer, maxlength, "Portuguese");
        case LANGID_BRAZILIAN: strcopy(buffer, maxlength, "Portuguese (Brazil)");
        case LANGID_RUSSIAN: strcopy(buffer, maxlength, "Russian");
        default: 
        {
            char langName[32];
            Lang_GetLanguageName(langId, langName, sizeof(langName));
            strcopy(buffer, maxlength, langName);
        }
    }
}
```

## Ejemplo 4: Sistema de Logros Localizados

### Plugin: Achievement System
```sourcepawn
enum struct Achievement
{
    char identifier[32];        // ID único del logro
    char titleKey[64];         // Clave de título Valve o personalizada
    char descKey[64];          // Clave de descripción
    bool isUnlocked[MAXPLAYERS + 1]; // Estado por jugador
    int progress[MAXPLAYERS + 1];    // Progreso actual
    int requirement;           // Requerimiento para desbloquear
}

#define MAX_ACHIEVEMENTS 20
Achievement g_Achievements[MAX_ACHIEVEMENTS];
int g_AchievementCount = 0;

public void OnPluginStart()
{
    // Inicializar Localizer
    g_Localizer = new Localizer();
    g_Localizer.Delegate_InitCompleted(OnPhrasesReady);
    
    RegConsoleCmd("sm_logros", CMD_Achievements, "Ver logros disponibles");
    RegConsoleCmd("sm_achievements", CMD_Achievements, "View available achievements");
    
    LoadAchievements();
    
    // Hooks de eventos para logros
    HookEvent("player_death", Event_PlayerDeath_Achievement);
    HookEvent("heal_success", Event_HealSuccess_Achievement);
    HookEvent("revive_success", Event_ReviveSuccess_Achievement);
}

public void OnPhrasesReady()
{
    // Localizer está listo para usar traducciones Valve
    LogMessage("Sistema de localización inicializado correctamente");
}

void LoadAchievements()
{
    g_AchievementCount = 0;
    
    // Logro: Eliminaciones
    strcopy(g_Achievements[g_AchievementCount].identifier, 32, "killer");
    strcopy(g_Achievements[g_AchievementCount].titleKey, 64, "achievement_killer_title");
    strcopy(g_Achievements[g_AchievementCount].descKey, 64, "achievement_killer_desc");
    g_Achievements[g_AchievementCount].requirement = 100;
    g_AchievementCount++;
    
    // Logro: Curandero
    strcopy(g_Achievements[g_AchievementCount].identifier, 32, "medic");
    strcopy(g_Achievements[g_AchievementCount].titleKey, 64, "achievement_medic_title");
    strcopy(g_Achievements[g_AchievementCount].descKey, 64, "achievement_medic_desc");
    g_Achievements[g_AchievementCount].requirement = 50;
    g_AchievementCount++;
    
    // Logro: Explorador (completar todas las campañas)
    strcopy(g_Achievements[g_AchievementCount].identifier, 32, "explorer");
    strcopy(g_Achievements[g_AchievementCount].titleKey, 64, "achievement_explorer_title");
    strcopy(g_Achievements[g_AchievementCount].descKey, 64, "achievement_explorer_desc");
    g_Achievements[g_AchievementCount].requirement = 14; // 14 campañas oficiales
    g_AchievementCount++;
}

public Action Event_PlayerDeath_Achievement(Event event, const char[] name, bool dontBroadcast)
{
    int attacker = GetClientOfUserId(event.GetInt("attacker"));
    int victim = GetClientOfUserId(event.GetInt("userid"));
    
    if (attacker > 0 && attacker <= MaxClients && attacker != victim)
    {
        UpdateAchievementProgress(attacker, "killer", 1);
    }
    
    return Plugin_Continue;
}

public Action Event_HealSuccess_Achievement(Event event, const char[] name, bool dontBroadcast)
{
    int client = GetClientOfUserId(event.GetInt("userid"));
    if (client > 0 && client <= MaxClients)
    {
        UpdateAchievementProgress(client, "medic", 1);
    }
    
    return Plugin_Continue;
}

void UpdateAchievementProgress(int client, const char[] achievementId, int increment)
{
    int achievementIndex = FindAchievementIndex(achievementId);
    if (achievementIndex == -1)
        return;
    
    if (g_Achievements[achievementIndex].isUnlocked[client])
        return; // Ya desbloqueado
    
    g_Achievements[achievementIndex].progress[client] += increment;
    
    // Verificar si se desbloqueó
    if (g_Achievements[achievementIndex].progress[client] >= 
        g_Achievements[achievementIndex].requirement)
    {
        UnlockAchievement(client, achievementIndex);
    }
    else
    {
        // Mostrar progreso
        ShowAchievementProgress(client, achievementIndex);
    }
}

void UnlockAchievement(int client, int achievementIndex)
{
    g_Achievements[achievementIndex].isUnlocked[client] = true;
    
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    // Obtener título y descripción localizados
    char title[128], description[256];
    GetLocalizedAchievementText(client, achievementIndex, title, sizeof(title), 
                               description, sizeof(description));
    
    // Anuncio personalizado por idioma
    if (Lang_IsSpanishVariant(clientLang))
    {
        PrintToChatAll("\x04¡%N ha desbloqueado el logro \x03%s\x04!", client, title);
        PrintToChat(client, "\x01%s", description);
        PrintToChat(client, "\x05¡Felicitaciones por tu logro!");
    }
    else if (Lang_IsFrench(clientLang))
    {
        PrintToChatAll("\x04%N a débloqué le succès \x03%s\x04!", client, title);
        PrintToChat(client, "\x01%s", description);
        PrintToChat(client, "\x05Félicitations pour votre réussite!");
    }
    else
    {
        PrintToChatAll("\x04%N unlocked achievement \x03%s\x04!", client, title);
        PrintToChat(client, "\x01%s", description);
        PrintToChat(client, "\x05Congratulations on your achievement!");
    }
    
    // Efectos adicionales (sonido, etc.)
    EmitSoundToClient(client, "ui/achievement_earned.wav");
}

void ShowAchievementProgress(int client, int achievementIndex)
{
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    char title[128];
    GetLocalizedAchievementTitle(client, achievementIndex, title, sizeof(title));
    
    int current = g_Achievements[achievementIndex].progress[client];
    int required = g_Achievements[achievementIndex].requirement;
    
    float percentage = (float(current) / float(required)) * 100.0;
    
    if (Lang_IsSpanishVariant(clientLang))
    {
        PrintHintText(client, "Progreso de logro:\n%s\n%d/%d (%.1f%%)", 
                     title, current, required, percentage);
    }
    else
    {
        PrintHintText(client, "Achievement progress:\n%s\n%d/%d (%.1f%%)", 
                     title, current, required, percentage);
    }
}

public Action CMD_Achievements(int client, int args)
{
    if (!client)
        return Plugin_Handled;
    
    ShowAchievementMenu(client);
    return Plugin_Handled;
}

void ShowAchievementMenu(int client)
{
    Menu menu = new Menu(Menu_AchievementHandler);
    
    if (Lang_IsSpanishVariant(Lang_GetClientLanguage(client)))
    {
        menu.SetTitle("Tus Logros:");
    }
    else
    {
        menu.SetTitle("Your Achievements:");
    }
    
    for (int i = 0; i < g_AchievementCount; i++)
    {
        char title[128], menuItem[192];
        GetLocalizedAchievementTitle(client, i, title, sizeof(title));
        
        if (g_Achievements[i].isUnlocked[client])
        {
            Format(menuItem, sizeof(menuItem), "%s", title);
        }
        else
        {
            int current = g_Achievements[i].progress[client];
            int required = g_Achievements[i].requirement;
            Format(menuItem, sizeof(menuItem), "%s (%d/%d)", title, current, required);
        }
        
        char indexStr[8];
        IntToString(i, indexStr, sizeof(indexStr));
        menu.AddItem(indexStr, menuItem);
    }
    
    menu.Display(client, MENU_TIME_FOREVER);
}

public int Menu_AchievementHandler(Menu menu, MenuAction action, int param1, int param2)
{
    switch (action)
    {
        case MenuAction_Select:
        {
            char indexStr[8];
            menu.GetItem(param2, indexStr, sizeof(indexStr));
            int achievementIndex = StringToInt(indexStr);
            
            ShowAchievementDetails(param1, achievementIndex);
        }
        case MenuAction_End:
        {
            delete menu;
        }
    }
    
    return 0;
}

void ShowAchievementDetails(int client, int achievementIndex)
{
    char title[128], description[256];
    GetLocalizedAchievementText(client, achievementIndex, title, sizeof(title), 
                               description, sizeof(description));
    
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    if (g_Achievements[achievementIndex].isUnlocked[client])
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            PrintToChat(client, "\x04Logro Desbloqueado:");
        }
        else
        {
            PrintToChat(client, "\x04Achievement Unlocked:");
        }
    }
    else
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            PrintToChat(client, "\x04Logro Bloqueado:");
        }
        else
        {
            PrintToChat(client, "\x04Achievement Locked:");
        }
    }
    
    PrintToChat(client, "\x03%s", title);
    PrintToChat(client, "\x01%s", description);
    
    if (!g_Achievements[achievementIndex].isUnlocked[client])
    {
        int current = g_Achievements[achievementIndex].progress[client];
        int required = g_Achievements[achievementIndex].requirement;
        
        if (Lang_IsSpanishVariant(clientLang))
        {
            PrintToChat(client, "\x05Progreso: %d/%d", current, required);
        }
        else
        {
            PrintToChat(client, "\x05Progress: %d/%d", current, required);
        }
    }
}

void GetLocalizedAchievementText(int client, int achievementIndex, char[] title, int titleLen, 
                                char[] description, int descLen)
{
    GetLocalizedAchievementTitle(client, achievementIndex, title, titleLen);
    
    // Para descripciones, usar traducción personalizada o fallback
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    if (StrEqual(g_Achievements[achievementIndex].identifier, "killer"))
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            strcopy(description, descLen, "Elimina 100 infectados especiales");
        }
        else
        {
            strcopy(description, descLen, "Kill 100 special infected");
        }
    }
    else if (StrEqual(g_Achievements[achievementIndex].identifier, "medic"))
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            strcopy(description, descLen, "Cura a otros supervivientes 50 veces");
        }
        else
        {
            strcopy(description, descLen, "Heal other survivors 50 times");
        }
    }
    else if (StrEqual(g_Achievements[achievementIndex].identifier, "explorer"))
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            strcopy(description, descLen, "Completa todas las campañas oficiales");
        }
        else
        {
            strcopy(description, descLen, "Complete all official campaigns");
        }
    }
}

void GetLocalizedAchievementTitle(int client, int achievementIndex, char[] title, int maxlength)
{
    LanguageID clientLang = Lang_GetClientLanguage(client);
    
    if (StrEqual(g_Achievements[achievementIndex].identifier, "killer"))
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            strcopy(title, maxlength, "Exterminador");
        }
        else if (Lang_IsFrench(clientLang))
        {
            strcopy(title, maxlength, "Exterminateur");
        }
        else
        {
            strcopy(title, maxlength, "Exterminator");
        }
    }
    else if (StrEqual(g_Achievements[achievementIndex].identifier, "medic"))
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            strcopy(title, maxlength, "Médico de Campo");
        }
        else if (Lang_IsFrench(clientLang))
        {
            strcopy(title, maxlength, "Médecin de Terrain");
        }
        else
        {
            strcopy(title, maxlength, "Field Medic");
        }
    }
    else if (StrEqual(g_Achievements[achievementIndex].identifier, "explorer"))
    {
        if (Lang_IsSpanishVariant(clientLang))
        {
            strcopy(title, maxlength, "Explorador Veterano");
        }
        else if (Lang_IsFrench(clientLang))
        {
            strcopy(title, maxlength, "Explorateur Vétéran");
        }
        else
        {
            strcopy(title, maxlength, "Veteran Explorer");
        }
    }
}

int FindAchievementIndex(const char[] identifier)
{
    for (int i = 0; i < g_AchievementCount; i++)
    {
        if (StrEqual(g_Achievements[i].identifier, identifier))
            return i;
    }
    return -1;
}
```

## Mejores Prácticas de Integración

### 1. Inicialización Correcta
```sourcepawn
public void OnPluginStart()
{
    // ORDEN IMPORTANTE:
    // 1. Inicializar Localizer primero
    g_Localizer = new Localizer();
    g_Localizer.Delegate_InitCompleted(OnPhrasesReady);
    
    // 2. Verificar dependencias
    if (!LibraryExists("left4dhooks"))
    {
        SetFailState("Requiere Left4DHooks");
    }
    
    // 3. Registrar comandos y hooks
    RegConsoleCmd("sm_comando", CMD_Handler, "Descripción");
    
    // 4. Configurar timers si es necesario
    CreateTimer(1.0, Timer_CheckReady, _, TIMER_REPEAT | TIMER_FLAG_NO_MAPCHANGE);
}

public void OnPhrasesReady()
{
    // Localizer está listo para usar traducciones Valve
    LogMessage("Sistema de localización inicializado correctamente");
}

public void OnAllPluginsLoaded()
{
    // Verificar que Localizer esté listo
    if (!g_Localizer.IsReady())
    {
        LogError("Localizer no está listo - algunas traducciones pueden fallar");
    }
}
```

### 2. Manejo de Errores Robusto
```sourcepawn
bool SafeGetLocalizedName(int client, const char[] mapCode, char[] buffer, int maxlength)
{
    // Validar parámetros
    if (client < 1 || client > MaxClients || !IsClientInGame(client))
    {
        strcopy(buffer, maxlength, "Invalid Client");
        return false;
    }
    
    if (strlen(mapCode) == 0)
    {
        strcopy(buffer, maxlength, "Invalid Map Code");
        return false;
    }
    
    // Intentar obtener nombre localizado
    if (Campaign_GetLocalizedNameFromMapCode(mapCode, client, buffer, maxlength, g_Localizer))
    {
        return true;
    }
    
    // Fallback con información útil
    Format(buffer, maxlength, "Map: %s", mapCode);
    return false;
}
```

### 3. Optimización de Performance
```sourcepawn
// Cache para evitar procesamiento repetitivo
StringMap g_MapNameCache;
StringMap g_ClientLanguageCache;

public void OnPluginStart()
{
    g_MapNameCache = new StringMap();
    g_ClientLanguageCache = new StringMap();
    
    // Limpiar cache periódicamente
    CreateTimer(300.0, Timer_ClearCache, _, TIMER_REPEAT);
}

public Action Timer_ClearCache(Handle timer)
{
    g_MapNameCache.Clear();
    g_ClientLanguageCache.Clear();
    return Plugin_Continue;
}
```

## Debugging y Testing

### Sistema de Debug Unificado
```sourcepawn
ConVar g_cvDebugLevel;

public void OnPluginStart()
{
    g_cvDebugLevel = CreateConVar("sm_localization_debug", "0", 
                                 "Debug level (0=Off, 1=Basic, 2=Verbose)", 
                                 FCVAR_DONTRECORD, true, 0.0, true, 2.0);
}

void DebugLog(int level, const char[] format, any ...)
{
    if (g_cvDebugLevel.IntValue < level)
        return;
    
    char buffer[512];
    VFormat(buffer, sizeof(buffer), format, 3);
    
    LogMessage("[Localization Debug L%d] %s", level, buffer);
}

// Uso:
DebugLog(1, "Cliente %N conectado con idioma %s", client, clientLang);
DebugLog(2, "Procesando mapa %s -> campaña %s", mapCode, campaignCode);
```

---

Estos ejemplos muestran cómo integrar eficientemente ambas librerías para crear sistemas completos y robustos de localización en Left 4 Dead 2. Cada ejemplo incluye manejo de errores, optimizaciones de performance y soporte completo para múltiples idiomas.

# Changelog — Aeon Panel

Todas las novedades destacables de Aeon (panel Pterodactyl con soporte extendido para Minecraft Bedrock).

El formato sigue [Keep a Changelog](https://keepachangelog.com/es/1.0.0/).

## [v2.0] — 2026-07

Hito: sistema completo de gestión de jugadores Bedrock con persistencia.

### Añadido
- **Gestión de jugadores en vivo** (pestaña Players): lista de jugadores conectados en tiempo real vía la consola del servidor, con contador de conectados/máximo (ej. 2/10).
- **Roster persistente de jugadores**: todos los jugadores que se han conectado quedan registrados (nombre, XUID, última conexión), con persistencia entre sesiones.
- **Estado visual en línea/desconectado**: los jugadores conectados aparecen primero y resaltados; los desconectados debajo, con su última conexión ("hace 5 min").
- **Gestión de operadores por nombre**: hacer/quitar operador desde la lista de jugadores, mostrando el nombre en vez del XUID.
- **Acciones rápidas**: añadir a lista blanca y gestionar operadores directamente desde cada jugador.
- **Contador de jugadores en la consola**: bloque "Jugadores" (X/10) junto a CPU/RAM/Disco, solo para servidores Bedrock.

## [v1.9] — 2026-07

Integración con CurseForge para addons de Bedrock.

### Añadido
- **Buscar e instalar addons desde CurseForge** directamente en la pestaña Addons.
- **Addons populares al abrir** y filtros por categoría (Resource Packs, Worlds, Scenarios).
- **Ordenamiento**: populares, nuevos, actualizados, relevancia.
- **Paginación** ("Cargar más") para explorar más resultados.
- **Descarga manual**: para addons cuyo autor no permite instalación automática vía API, enlace directo a CurseForge.

### Notas
- La API de CurseForge prioriza contenido de Java; algunos addons de Bedrock de la web no están indexados en la API y solo pueden instalarse mediante subida manual.

## [v1.8] — 2026-07

Edición NBT de mundos y mejoras mayores de addons.

### Añadido
- **Edición NBT del level.dat** de mundos Bedrock (lectura y escritura, formato little-endian).
- **Gestión de experimentos** del mundo (Beta APIs, Molang, etc.).
- **Control de cheats** sincronizado entre NBT y server.properties.
- **Agrupación de addons**: los packs de comportamiento y recursos de un mismo addon se muestran juntos, con activación en grupo.
- **Iconos reales de packs** (pack_icon.png) en la lista de addons.
- **Soporte de .mcaddon con packs anidados**: instala correctamente archivos que contienen varios .mcpack dentro.

### Corregido
- Detección de packs del sistema (vanilla) para no ocultar addons legítimos con nombre de clave de traducción.
- Eliminación de packs con caracteres especiales en el nombre.

## [v1.7] — 2026-06

Configuración de servidor y gestión de mundos.

### Añadido
- **Pestaña Server Settings** estilo Minecraft Bedrock, con controles de alternancia y opciones avanzadas.
- **Renombrar mundos** mediante la edición del nombre en el level.dat.
- **Descarga de mundos** como archivo .mcworld real.

## [v1.6] — 2026-06

Bases de la gestión de addons y mundos Bedrock.

### Añadido
- **Pestañas Bedrock** (Addons, Worlds, Players, Server Settings) visibles solo en servidores Bedrock.
- **Subida e instalación de packs** (.mcpack, .mcaddon).
- **Activación/desactivación de packs** por mundo.
- **Forzar packs a jugadores** (texturepack-required).
- **Subida y descarga de mundos**.

## Bases del proyecto (antes de v1.6)

Antes del registro por versiones, Aeon se construyó como una versión personalizada de Pterodactyl con un conjunto de mejoras centradas en Minecraft Java. Estas funcionalidades ya formaban parte del panel:

### Minecraft Java
- **Instalador de modpacks con un clic** (pestaña Modpacks): instala modpacks de CurseForge automáticamente, gestionando la descarga, el loader (NeoForge), los mods, el EULA y el mundo inicial.
- **Pestaña Players (Java)**: visualización de vida, hambre, experiencia e inventario de los jugadores, con iconos reales de items (vanilla y de mods). Los iconos de mods se generan automáticamente al instalar un modpack.

### General
- **Rebrand del panel a "Aeon"**, como versión personalizada e independiente de Pterodactyl.
- **Subdominios automáticos** (pestaña Subdomains): integración con Cloudflare para crear subdominios tipo `tuservidor.tudominio.com`.
- **Pestañas inteligentes**: las herramientas específicas de cada tipo de servidor (Java, Bedrock) solo aparecen en los servidores correspondientes.
- **Distribución como imagen Docker**: todo el código del panel viene horneado en la imagen, de modo que la instalación se reduce a levantar el compose.

### Infraestructura
- Configuración de despliegue con Pterodactyl Panel + Wings sobre Docker.
- Script de configuración del host (`apply-host-config.sh`) para firewall y ajustes de red.

---

Aeon es un proyecto personal basado en [Pterodactyl](https://pterodactyl.io/), enfocado en mejorar la experiencia de administración de servidores Minecraft (Java y Bedrock).

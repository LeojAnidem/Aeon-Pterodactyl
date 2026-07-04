# Aeon Pterodactyl — Guía de instalación

Bienvenido. Esta es una versión personalizada del panel Pterodactyl con mejoras
para servidores de Minecraft **Java y Bedrock**: instalador de modpacks con un
clic, gestión completa de addons y mundos Bedrock, pestaña de jugadores,
subdominios automáticos y más. Consulta el [CHANGELOG](CHANGELOG.md) para ver
todas las novedades por versión.

Todo el código del panel viene **horneado en la imagen Docker**, así que la
instalación es muy sencilla: levantas el compose, creas tu usuario, configuras
el firewall y conectas Wings.

---

## Requisitos previos

- Una máquina con **Docker** y **Docker Compose** instalados
- **UFW** (firewall) instalado: `sudo apt install ufw`
- Acceso `sudo` en la máquina
- (Opcional) Un dominio si quieres subdominios automáticos

---

## Paso 1 — Levantar el panel

1. Crea una carpeta para el panel y coloca dentro el archivo `docker-compose.yml`.

2. **Edita el `docker-compose.yml`** y ajusta estas variables del servicio `panel`:
   - `APP_URL`: la dirección donde accederás al panel (ej. `http://192.168.1.100:4080`)
   - `APP_TIMEZONE`: tu zona horaria (ej. `America/Mexico_City`)
   - Las contraseñas de la base de datos (`MYSQL_PASSWORD`, `DB_PASSWORD`, etc.)
     — cámbialas por unas tuyas, deben coincidir entre los servicios.
   - **IMPORTANTE:** genera tu propia `APP_KEY` (ver Paso 1.1)

3. Levanta los contenedores:
   ```bash
   docker compose up -d
   ```

4. Espera ~30 segundos a que el panel arranque. Verifica con:
   ```bash
   docker ps
   ```

### Paso 1.1 — Generar tu APP_KEY (importante para la seguridad)

La `APP_KEY` cifra los datos sensibles. **No uses la del ejemplo.** Genera una:

```bash
docker exec aeon_panel php artisan key:generate --show
```

Copia el valor que empieza con `base64:` y pégalo en `APP_KEY` del compose. Luego
reinicia: `docker compose up -d`.

> ⚠️ Si cambias la APP_KEY después de tener datos, esos datos cifrados se pierden.
> Por eso, genérala ANTES de crear usuarios o servidores.

---

## Paso 2 — Crear tu primer usuario (acceso al panel)

El panel no trae usuarios por defecto. Crea tu cuenta de administrador con:

```bash
docker exec -it aeon_panel php artisan p:user:make
```

El asistente te preguntará:
- **Is this user an administrator?** → escribe `yes` (para tener acceso total)
- **Email Address** → tu correo (será tu login)
- **Username** → un nombre de usuario
- **First Name / Last Name** → tu nombre
- **Password** → una contraseña segura (no se muestra al escribir, es normal)

> 💡 TIP: El `-it` es importante (permite la interacción). Si lo olvidas, el
> comando no te dejará escribir las respuestas.

Una vez creado, abre tu navegador en la `APP_URL` que configuraste
(ej. `http://192.168.1.100:4080`) e inicia sesión con ese email y contraseña.

---

## Paso 3 — Configurar el firewall y los puertos

Los servidores de juego necesitan que el firewall permita el tráfico externo
hacia los contenedores. El script `apply-host-config.sh` lo hace por ti.

### 3.1 — Elegir tu rango de puertos

Decide qué rango de puertos usarán tus servidores de juego. Por defecto usamos
**7000-31000** (cubre cientos de servidores). Puedes dejarlo así o ajustarlo.

Este rango debe coincidir en TRES lugares:
1. El firewall (lo aplica el script)
2. Las allocations del panel (Paso 5)
3. El reenvío de puertos de tu router (si quieres acceso desde internet)

### 3.2 — Aplicar las reglas del firewall + automatización

```bash
sudo bash apply-host-config.sh
```

Esto aplica:
- Reglas UFW que permiten el acceso externo a los servidores de juego
- Dos timers que mantienen los iconos y archivos sincronizados automáticamente

> El script **auto-detecta** tu subred Docker y abre los puertos del panel, del
> daemon de Wings y del rango de juego. Ejecútalo DESPUÉS de crear el nodo. Si
> algo no conecta, vuelve a correrlo.

### 3.3 — Reenvío de puertos en el router (solo para acceso desde internet)

Si quieres que gente de fuera de tu red se conecte, entra a la configuración de
tu router y reenvía tu rango de puertos (ej. 7000-31000) **TCP y UDP** hacia la
IP local de tu máquina. Los juegos NO pasan por túneles web; necesitan esto.

---

## Paso 4 — Configurar Wings (el demonio que corre los servidores)

Wings ejecuta los servidores de juego. El panel y Wings se vinculan con un
**token** que genera tu panel. Sigue el orden con cuidado: los detalles importan.

### 4.1 — Crear las carpetas de Wings (una vez)

Wings necesita estas tres rutas en el host (el compose las monta con ruta
idéntica; es un requisito de Docker):

```bash
sudo mkdir -p /var/lib/pterodactyl/volumes /tmp/pterodactyl /run/wings
```

### 4.2 — Crear un Node en el panel

1. **Admin** → **Locations** → crea una (ej. nombre `local`) si no hay ninguna.
2. **Admin** → **Nodes** → **Create New**:
   - **Name:** `principal`
   - **Location:** la que creaste
   - **FQDN:** tu **IP LAN** (la misma de `APP_URL`, ej. `192.168.1.100`)
   - **Communicate Over SSL:** **desactivado** (usamos HTTP)
   - **Behind Proxy:** No
   - **Daemon Port:** `8080` (debe coincidir con el puerto de Wings del compose)
   - **Memory / Disk:** valores generosos (ej. 8000 / 50000)
3. Guarda.

### 4.3 — Instalar la configuración de Wings

1. Abre el Node → pestaña **Configuration**. Copia el bloque YAML.
2. Guárdalo en `./data/wings/config.yml` (junto a tu compose):
   ```bash
   nano data/wings/config.yml
   ```
   Pega, guarda (Ctrl+O, Enter, Ctrl+X).

3. **Verifica dos valores en ese config** (importantes):
   - `api: port:` debe ser `8080` (coincide con el compose).
   - `remote:` debe ser tu **APP_URL con la IP LAN** (ej. `http://192.168.1.100:4080`),
     NO un nombre interno. Esto es clave para que la conexión y el CORS funcionen.

### 4.4 — Reiniciar Wings

```bash
docker compose restart wings
docker compose logs wings --tail 8
```

Busca en los logs `sftp server listening` y `processing servers returned by the
API`. Si aparecen, Wings conectó. En el panel, el Node debe ponerse **verde**.

> ⚠️ **Si el corazón sale rojo o el "About" del nodo no carga**, casi siempre es
> el firewall bloqueando la comunicación entre contenedores. El comando del Paso
> 3.2 (`apply-host-config.sh`) abre los puertos necesarios (panel y daemon).
> Asegúrate de haberlo ejecutado. Si lo ejecutaste antes de crear el nodo,
> vuelve a correrlo.

## Paso 5 — Crear las allocations (puertos para servidores)

Para que puedas crear servidores, el Node necesita "allocations" (puertos
disponibles).

1. En el panel: **Admin** → **Nodes** → tu Node → pestaña **Allocations**
2. En **Assign New Allocations**:
   - **IP:** `0.0.0.0` (escucha en todas las interfaces)
   - **Ports:** tu rango, ej. `30000-30050` (deben estar dentro del rango del
     firewall del Paso 3)
3. Click en **Assign**.

Ahora puedes crear servidores que usarán esos puertos.

---

## ¡Listo! Crear tu primer servidor

1. **Admin** → **Servers** → **Create New**
2. Elige el dueño, el Node, una allocation, los recursos (RAM, CPU, disco)
3. Elige el **Egg** (tipo de servidor: Minecraft Java, Bedrock, etc.)
4. Crea y espera a que se instale.

Para Minecraft Java verás las pestañas extra: **Players, Mods, Modpacks,
Worlds, Subdomains**, etc. Para instalar un modpack con un clic, ve a la pestaña
**Modpacks**.

---

## Características incluidas

> 📋 Consulta el historial completo de versiones y novedades en el [**CHANGELOG**](CHANGELOG.md).

Además de las funciones de Minecraft Java, Aeon incluye un conjunto completo de herramientas para **Minecraft Bedrock**: gestión de addons (con CurseForge), mundos (con edición NBT), jugadores en vivo y configuración del servidor. Todo el detalle está en el [CHANGELOG](CHANGELOG.md).

- **Modpacks con un clic** (pestaña Modpacks): instala CurseForge modpacks
  automáticamente (descarga, NeoForge, mods, EULA, mundo limpio).
- **Pestaña Players**: vida, hambre, experiencia, inventario con iconos reales
  de items (vanilla y de mods). Los iconos de mods se generan solos al instalar
  un modpack.
- **Subdominios automáticos** (pestaña Subdomains): si configuras Cloudflare,
  crea subdominios tipo `tuserver.tudominio.com`.
- **Pestañas inteligentes**: las herramientas de Minecraft Java solo aparecen en
  servidores de Minecraft Java.

---

## Solución de problemas

- **No puedo acceder al panel:** revisa `APP_URL` y que el puerto (4080) esté
  libre y abierto.
- **El Node sale offline:** revisa el puerto del daemon de Wings y el token.
- **No me conecto a un servidor desde internet:** revisa el reenvío de puertos
  del router (TCP+UDP) y que `apply-host-config.sh` se ejecutó.
- **Los iconos de un modpack no aparecen:** espera ~2 minutos (el watcher los
  genera tras instalar el modpack) y recarga la pestaña Players.

---

## Notas

- Esta es una versión personalizada e independiente. Si algo falla, es
  responsabilidad de esta versión, no de los proyectos en los que se basa.
- Trata tu `docker-compose.yml` como secreto (contiene contraseñas).

# Aeon Pterodactyl — Guía de instalación

Bienvenido. Esta es una versión personalizada del panel Pterodactyl con mejoras
para servidores de Minecraft: instalador de modpacks con un clic, pestaña de
jugadores (vida, hambre, inventario con iconos), subdominios automáticos y más.

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

> ⚠️ Revisa que la **subred de Wings** en el script coincida con la tuya. Por
> defecto es `172.27.0.0/16`. Si tu red Docker de Wings es distinta, edita la
> variable `WINGS_SUBNET` en el script antes de ejecutarlo.

### 3.3 — Reenvío de puertos en el router (solo para acceso desde internet)

Si quieres que gente de fuera de tu red se conecte, entra a la configuración de
tu router y reenvía tu rango de puertos (ej. 7000-31000) **TCP y UDP** hacia la
IP local de tu máquina. Los juegos NO pasan por túneles web; necesitan esto.

---

## Paso 4 — Configurar Wings (el demonio que corre los servidores)

Wings es el componente que ejecuta los servidores de juego. El panel y Wings se
vinculan con un **token único** que genera tu panel.

### 4.1 — Crear un Node en el panel

1. En el panel, ve a **Admin** (icono de configuración) → **Nodes** → **Create New**
2. Rellena:
   - **Name:** un nombre (ej. "Nodo Principal")
   - **FQDN / IP:** la IP o dominio de tu máquina
   - **Daemon Port:** normalmente `8080` (o el que uses)
   - El resto, valores por defecto está bien para empezar
3. Guarda.

### 4.2 — Obtener la configuración de Wings

1. Abre el Node que creaste → pestaña **Configuration**
2. Verás un bloque de configuración YAML. Cópialo.
3. Pégalo en el archivo de configuración de Wings de tu máquina, normalmente en:
   ```
   /etc/pterodactyl/config.yml
   ```
   (o la ruta que montaste para Wings en el compose)

### 4.3 — Reiniciar Wings

```bash
docker compose restart wings
```

Verifica en el panel que el Node aparezca **en línea** (un indicador verde).

> 💡 TIP: Si el Node sale "offline", revisa que el puerto del daemon (8080) esté
> abierto en el firewall y que la IP/FQDN del Node sea correcta.

---

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

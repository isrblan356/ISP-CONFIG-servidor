# ISP CONFIG — servidor de sincronización (PHP 8 + SQLite)

API + panel web que la APK usa para bajar zonas/APs y subir/bajar nodos.
Cero dependencias: solo PHP 8 con `pdo_sqlite` (si no lo tienes, lo instala solo).

## Instalar en la otra PC (todo: código + base de datos)

**Opción A — con git** (PowerShell o CMD en la PC destino):

```powershell
git clone https://github.com/isrblan356/ISP-CONFIG-servidor.git; cd ISP-CONFIG-servidor; .\instalar.bat
```

**Opción B — sin git, una sola línea** (PowerShell):

```powershell
iwr https://github.com/isrblan356/ISP-CONFIG-servidor/archive/refs/heads/main.zip -OutFile s.zip; Expand-Archive s.zip; cd ISP-CONFIG-servidor-main; .\instalar.bat
```

`instalar.bat` hace todo solo:

1. Busca PHP (PATH, junto al archivo, o `%ProgramFiles%\PHP`); si no lo encuentra
   lo instala con `winget` (PHP 8.3).
2. Crea `php-isp-config.ini` con `pdo_sqlite` + `sqlite3` activos.
3. Siembra `data/isp.sqlite` con las 8 zonas + 259 APs (solo la primera vez).
4. Arranca el servidor en `0.0.0.0:8080` en una ventana **minimizada**
   (`ISP CONFIG` — no la cierres, esa es la que sirve).
5. **Abre el navegador** en el panel web `http://localhost:8080/admin`.

**Panel web** (`/admin`): usuario `admin`, contraseña `admin` (cámbiala en
*Ajustes*). Ahí gestionas **nodos con coordenadas**, **APs**, **zonas**,
**firmware**, **técnicos con su IP** (el buscador te dice si una IP está libre o
quién la tiene) y **dispositivos con la APK** (a cada uno le asignas su técnico
e IP desde el desplegable de la fila). Si no entraras con `admin`, borra
`data\isp.sqlite` y vuelve a ejecutar: se re-siembra con `admin`/`admin`.

**Firewall**: cuando Windows pregunte por `php.exe` → permite en *Red privada*.

**Detener**: cierra la ventana `ISP CONFIG` minimizada (o `Ctrl+C` ahí).

**Sin winget**: baja el zip *x64* de <https://windows.php.net/download/>,
descomprime en `C:\php` (o junto a este archivo) y vuelve a ejecutar
`instalar.bat`; si prefieres a mano: `extension=pdo_sqlite` y `extension=sqlite3`
en `php.ini` y arranca `iniciar.bat`.

## Otras formas de correrlo

```powershell
.\iniciar.bat    # solo arranca (si PHP ya está y BD ya existe)
```

WSL / Linux:

```bash
sudo apt install php-cli php-sqlite3     # una vez
php scripts/seed.php                     # crea data/isp.sqlite
php -S 0.0.0.0:8080 -t public            # 0.0.0.0 para que el celular entre
```

Comprobación:

```bash
curl http://127.0.0.1:8080/health      # {"ok":true,...}
curl http://127.0.0.1:8080/zones
curl "http://127.0.0.1:8080/aps?zona=1"
curl http://127.0.0.1:8080/nodes
```

## Apuntar la APK

1. IP de la PC servidor: `ipconfig` (Windows) o `hostname -I` (Linux). Ej: `192.168.1.50`.
2. Celular y PC en la **misma WiFi**.
3. En la APK: pestaña **Admin** → contraseña → **Nube** → URL `http://192.168.1.50:8080`
   → *Activar sincronización* → **Probar conexión**.
4. Las zonas/APs bajan con el botón de recarga en *Zonas/APs*; los nodos con el
   botón ☁ en *Brújula*.

La APK trae `usesCleartextTraffic="true"`, así que el HTTP local funciona sin HTTPS.

## Escrituras con token (opcional, recomendado fuera de local)

```powershell
setx ISP_ADMIN_TOKEN "cambia-esto"     # reabre la consola
.\iniciar.bat
```

Con eso, `POST/PUT/DELETE /nodes` exigen la cabecera `X-Admin-Token`. En la APK:
Admin → Nube → *Token de escritura*.

## Subirlo a un hosting (más adelante)

- **Hosting PHP**: sube esta carpeta por FTP, apunta el dominio a `public/`,
  asegúrate de `AllowOverride All` (viene `.htaccess`), crea la BD con
  `php scripts/seed.php` desde la consola del hosting.
- **VPS**: nginx/Apache + PHP-FPM con la raíz en `public`; `data/` debe ser
  escribible por el usuario de PHP.

## Endpoints

| Método | Ruta | Uso |
|---|---|---|
| GET | `/health` | probar conexión desde la APK |
| GET | `/zones` | zonas (id, zona, network) |
| GET | `/aps?zona=1` | APs de una zona |
| GET | `/nodes` | nodos con coordenadas |
| POST/PUT | `/nodes` | crear/actualizar nodo (JSON) |
| DELETE | `/nodes?id=1` | borrar nodo |
| GET | `/stats` | conteos para el panel |
| GET | `/session` · POST · DELETE | sesión del panel (`admin`/`admin`) |
| POST | `/password` | cambiar contraseña (con sesión) |
| GET/POST/PUT/DELETE | `/tecnicos` | técnicos con su IP (`?ip=` → ¿libre?) |
| GET/POST/PUT/DELETE | `/firmware` | modelos/versiones de firmware |
| GET/POST/PUT/DELETE | `/dispositivos` | celulares con la APK (modelo, IP, técnico) |
| POST/PUT/DELETE | `/zones` | administrar zonas |
| GET | `/admin/` | panel web (HTML/JS/CSS) |

Errores: `{"error":"..."}` con código HTTP; la APK nunca borra lo local si la
respuesta no es `200` con datos válidos.

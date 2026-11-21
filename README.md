# Conectarse a la PC de Meme por Tailscale

Pasos para el Claude del hermano de Meme. La PC del hermano entra a la red de Tailscale de Meme para usar la tarjeta gráfica de Meme (NVIDIA RTX 5050 Laptop, 8 GB).

La llave de acceso no está en este repo. Meme se la pasa a su hermano por mensaje. Empieza con `tskey-auth-`, sirve para un solo equipo y vence el 28-11-2026. No la guardes en ningún repo ni la pegues en un lugar público.

PC de Meme en Tailscale

- Nombre `pc-meme`
- IP `100.91.193.93`

## 1. Instalar Tailscale

Windows, en PowerShell

```powershell
winget install --id Tailscale.Tailscale -e --accept-source-agreements --accept-package-agreements
```

Si no hay winget, el instalador está en https://tailscale.com/download/windows. Después usa la ruta completa, porque una consola abierta antes de instalar no ve el PATH nuevo.

```powershell
$ts = "C:\Program Files\Tailscale\tailscale.exe"
```

En macOS, `brew install --cask tailscale` y abrir la app una vez. En Linux, `curl -fsSL https://tailscale.com/install.sh | sh` y los comandos de abajo con `sudo tailscale ...`.

## 2. Entrar a la red de Meme con la llave

Si Tailscale es nuevo en esa PC

```powershell
& $ts up --auth-key=LA_LLAVE --hostname=pc-hermano
```

Si esa PC ya tenía Tailscale con otra cuenta, `up` no cambia de red. Usa `login`, que agrega la red de Meme como un perfil aparte y se pasa a ese.

```powershell
& $ts login --auth-key=LA_LLAVE --hostname=pc-hermano
```

Para volver después a la cuenta propia, `& $ts switch --list` y `& $ts switch <cuenta>`.

## 3. Comprobar

```powershell
& $ts status
& $ts ip -4
```

En `status` tiene que salir `pc-meme` con `100.91.193.93`. El segundo comando da la IP propia, que empieza con `100.`.

## 4. Ping hacia Meme

```powershell
& $ts ping pc-meme
```

Lo bueno es ver `pong from pc-meme (100.91.193.93) via ...`. Si dice `via DERP(mia)` va por un relay, funciona pero más lento. Si dice `via <ip>:<puerto>` la conexión es directa.

`tailscale ping` no depende del firewall de Windows. El `ping` normal puede no contestar si el firewall de la otra PC lo bloquea, y eso no es falla de Tailscale.

## 5. Lo que le devuelves a Meme

Dale a tu usuario este mensaje para que se lo mande a Meme, con sus datos reales

```text
Ya entré a tu Tailscale
Nombre pc-hermano
IP 100.x.x.x
tailscale ping pc-meme dio pong via ...
```

Con eso, desde la PC de Meme se hace `tailscale ping pc-hermano` y quedan probados los dos sentidos.

Si también quieren que el `ping` normal de Meme hacia esta PC conteste, en Windows y como administrador, solo si tu usuario lo aprueba

```powershell
New-NetFirewallRule -DisplayName "Ping por Tailscale" -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -RemoteAddress 100.64.0.0/10 -Action Allow
```

## 6. Si no conecta

- Una VPN como Cloudflare WARP bloquea a Tailscale. Hay que desconectarla, o excluir `100.64.0.0/10`, `192.200.0.0/24` y los dominios de `tailscale.com`.
- Si `status` se queda en offline, en Windows y como administrador `Restart-Service Tailscale`, y volver a mirar.
- Si la llave dice que venció o que ya se usó, Meme genera otra.

## 7. Usar la gráfica

Cuando el ping ya funcione en los dos sentidos, dile a Meme qué programa va a usar la gráfica (por ejemplo Ollama, ComfyUI o SSH) y en qué puerto. Meme lo abre en su PC solo para la red de Tailscale y te pasa la dirección, que va a ser `100.91.193.93` con ese puerto.

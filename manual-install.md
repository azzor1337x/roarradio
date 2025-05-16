
## Instalar um Editor de Texto

```
apk add nano
```

## Alterar Repositorio

```
rm -rf /etc/apk/repositories
nano /etc/apk/repositories
```

Conteúdo:
```
http://dl-cdn.alpinelinux.org/alpine/v3.21/main
http://dl-cdn.alpinelinux.org/alpine/v3.21/community
```

## Instalar Docker + Icecast

```
apk update
apk add docker icecast curl bash git
```

```
rc-update add docker boot && service docker start
rc-update add icecast boot && service icecast start
```

```
docker run hello-world
```

## Portainer

```
docker volume create portainer_data
docker run -d -p 9000:9000 -p 9443:9443 --name portainer \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

## Git Clone Radio

```
git clone https://github.com/azzor1337x/radio /home/radio
```

## Criar Stack do MeTube no Portainer

```
services:
  metube:
    image: ghcr.io/alexta69/metube
    container_name: metube
    restart: unless-stopped
    ports:
      - 8081:8081
    volumes:
      - /home/music:/downloads
```

## Criar Stack do Liquidsoap no Portainer

```
services:
  liquidsoap:
    image: savonet/liquidsoap:v2.3.2
    container_name: liquidsoap
    volumes:
      - /home/radio/liquidsoap/radio.liq:/etc/liquidsoap/radio.liq
      - /home/music:/home/music
    restart: unless-stopped
    command: [liquidsoap, /etc/liquidsoap/radio.liq]
    extra_hosts:
      - host.docker.internal:host-gateway
```

## Criar Stack do Radio no Portainer

```
services:
  radio:
    image: nginx:alpine
    container_name: radio
    ports:
      - 8001:80
    volumes:
      - /home/radio/site:/usr/share/nginx/html:ro
    restart: unless-stopped
```

## instalar Cloudflared

```
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -O /usr/bin/cloudflared
chmod +x /usr/bin/cloudflared
```

```
cloudflared --version
cloudflared tunnel login
```
Copiar a URL e logar em outro dispositivo

```
cloudflared tunnel create radio
```
Guardar o UUID

```
nano /root/.cloudflared/config.yml
```

Conteúdo:
```
tunnel: radio
credentials-file: /root/.cloudflared/<UUID>.json

ingress:

  - hostname: icecast.roarradio.site
    service: http://127.0.0.1:8000

  - hostname: roarradio.site
    service: http://127.0.0.1:8001

  - service: http_status:404
```
UUID.cfargotunnel.com

```
cloudflared tunnel run radio
```
Se funcionar, seguir com as etapas abaixo

```
nano /etc/init.d/cloudflared
```

Conteúdo:
```
#!/sbin/openrc-run

name="cloudflared"
description="Cloudflare Tunnel"
command="/usr/bin/cloudflared"
command_args="tunnel --config /root/.cloudflared/config.yml run radio"
pidfile="/var/run/cloudflared.pid"
command_background="yes"

output_log="/var/log/cloudflared.log"
error_log="/var/log/cloudflared.err"

depend() {
    need docker
}

start_pre() {
    ebegin "Aguardando container radio iniciar"
    until docker ps | grep -q radio; do
        sleep 2
    done
    eend 0
}
```

```
chmod +x /etc/init.d/cloudflared
rc-update add cloudflared default
```

```
rc-service cloudflared start
```

## File Browser

```
cd /
```

```
curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash
filebrowser -a 0.0.0.0 -p 8888 -r /
```

## Finalizar!

```
apk update && apk upgrade
```

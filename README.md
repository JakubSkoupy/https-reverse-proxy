Steps:
  Install docker
  Install docker-compose

  create docker-compose.yml file
  ```
version: '3'

services:
  reverse-proxy:

    # The official v3 Traefik docker image
    image: traefik:v3.1
    network_mode: host

    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock


      # Mount the conf volume
      - ${working_directory}/conf:/etc/traefik
```
make sure to follow the indentation

make a conf directory

add sites.yml to "conf"
```
http:
  services:
    app:
      loadBalancer:
        servers:
          - url: "http://localhost:8081"

  routers:
    router0:  
      service: app
      rule: PathPrefix(`/`)
      #rule: Host(`localhost`) || Host(`192.168.2.78`) 
      # TLS might need host to provide a certificate, only using PathPrefix rule might not work
      tls: {}
```
add traefik.yml to "conf"
```
entryPoints:
  web:
    address: ":80"   # HTTP port
  websecure:
    address: ":443"  # HTTPS port

providers:
  file:
    filename: /etc/traefik/sites.yml

log:
  level: TRACE

api:
    insecure: true
    dashboard: true
```


To start the proxy, run: "docker compose up -d"
To stop the proxy, run: "docker compose down"
You will probably need superuser privileges (use sudo)

You can verify the proxy is running using: "docker ps -l"

For errors look in "docker logs ${container name or hash}"

## About
Redirects communication from entry ports to another port. (In this case, ports `80` and `443` are redirected to `8081` where my app is running)

Useful for connecting to an app from the local network.


You can use the configuration in this repo, and change a few lines according to the readme. Each step explains what the parameters
mean, so you can follow the guide to edit an existing configuration.
```
├── conf
│   ├── sites.yml
│   └── traefik.yml
└── docker-compose.yml
```


## Steps
### 1. Install docker
[ldocker]: https://docs.docker.com/desktop/install/linux-install
[ldocker engine]: https://docs.docker.com/engine/install/
[ldocker-compose]: https://docs.docker.com/desktop/install/linux-install/

Install [docker][ldocker], [docker engine][ldocker engine] and [docker compose][ldocker-compose]

### 2. create docker-compose.yml file
1. Create a service
```
services:
  reverse-proxy:

    # The official v3 Traefik docker image
    image: traefik:v3.1
    network_mode: host
```
`image`: set to the traefik image


`network_mode`: set to `host`, and verify that the `host` network exists with:
```
docker network ls
```
2. Mount the docker socket and `conf` volumes
create a `conf` directory in the same directory as your `docker-compose.yml`
add the `volumes` section to the yml
```
services:
  reverse-proxy:

    # The official v3 Traefik docker image
    image: traefik:v3.1
    network_mode: host

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock


      # Mount the conf volume
      - ./conf:/etc/traefik
```
make a conf directory

### 3. create sites.yml
add `sites.yml` to `conf`
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
make sure the `services/app` matches with `routers/router0/service`
rule `PathPrefix(`/`)` should accept everything on the entry ports
`tls {}` (transport layer security) is needed for the https (port 443), without this, some ports will work, but not 443.
`tls` Without arguments makes traefik provide a self-signed certificate, that will not be trusted.

### 4. create traefik.yml
add `traefik.yml` to `conf` directory
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
`entryPoints`: specify the ports to redirect to the app

`providers`: path where `sites.yml` gets mounted

`log/level`: level of log information (`TRACE` is the most verbose)

`dashboard`: true / false: specifies if we want the traefik dashboard in our browser

`insecure`: uses port 8080. Might cause problems if the port 8080 is already reserved

### 5. run the docker compose

Start the docker daemon
on sytemD linux:
```
systemctl --user start docker
```
or to enable by default (--now starts the service)
```
systemctl --user enable docker --now
```

Ubuntu
```
sudo service docker start
```
To start the proxy, run:
```
docker compose up -d
```
-d runs docker as a background process

To stop the proxy, run:
```
docker compose down
```

**You will probably need superuser privileges (use sudo)**

You can verify the proxy is running using:
```
docker ps -l
```

For errors look in
```
docker logs [ container name or hash ] 
```

## Notes
Some browsers will find the https certificate suspicious

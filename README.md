<p align="center">
<img src="https://cwmkt.com.br/wp-content/uploads/2024/04/logo_github.png" width="240" />
<p align="center">Seja bem-vindo ao Guia de Instalação Docker Evolution API 🚀</p>
</p>
  
<p align="center"> 
<a href="https://hubconnect.top" target="_blank">👉 Participe da Comunidade HubConnect 👈</a>
</p>

<hr />
<hr />

### Caso não tenha Portainer e Traefix instalado, siga primeira etapa

<details>
<summary>Instalando Portainer e Traefix</summary>

### Atualizando Dependências

Atualize os repositórios do Ubuntu executando o seguinte comando:

```bash
sudo apt update && apt upgrade -y
```

----------------------------------------------------------------------------

**Instale o Docker em sua VPS**

```bash
sudo apt install docker.io -y
```

----------------------------------------------------------------------------

**Instalando Portainer**

```bash
docker swarm init
```

```bash
nano traefik.yml
```

```bash
version: "3.8"

services:

  traefik:
    image: traefik:2.11.1
    command:
      - "--api.dashboard=true"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=ecosystem_network"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.web.http.redirections.entrypoint.permanent=true"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencryptresolver.acme.email=contato@seudominio.com.br"
      - "--certificatesresolvers.letsencryptresolver.acme.storage=/etc/traefik/letsencrypt/acme.json"
      - "--log.level=DEBUG"
      - "--log.format=common"
      - "--log.filePath=/var/log/traefik/traefik.log"
      - "--accesslog=true"
      - "--accesslog.filepath=/var/log/traefik/access-log"
    deploy:
      placement:
        constraints:
          - node.role == manager
      labels:
        - "traefik.enable=true"
        - "traefik.http.middlewares.redirect-https.redirectscheme.scheme=https"
        - "traefik.http.middlewares.redirect-https.redirectscheme.permanent=true"
        - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
        - "traefik.http.routers.http-catchall.entrypoints=web"
        - "traefik.http.routers.http-catchall.middlewares=redirect-https@docker"
        - "traefik.http.routers.http-catchall.priority=1"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "traefik_certificates_volume:/etc/traefik/letsencrypt"
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 443
        published: 443
        mode: host
    networks:
      - ecosystem_network

volumes:
  traefik_certificates_volume:
    external: true
    name: traefik_certificates_volume

networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
 ```

```bash
nano portainer.yml
```

```bash
version: "3.8"

services:

  agent:
    image: portainer/agent:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - ecosystem_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:latest
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    volumes:
      - portainer_volume:/data
    networks:
      - ecosystem_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=ecosystem_network"
        - "traefik.http.routers.portainer.rule=Host(`seudominio.com.br`)"
        - "traefik.http.routers.portainer.entrypoints=websecure"
        - "traefik.http.routers.portainer.priority=1"
        - "traefik.http.routers.portainer.tls.certresolver=letsencryptresolver"
        - "traefik.http.routers.portainer.service=portainer"
        - "traefik.http.services.portainer.loadbalancer.server.port=9000"

networks:
  ecosystem_network:
    external: true
    attachable: true
    name: ecosystem_network

volumes:
  portainer_volume:
    external: true
    name: portainer_volume

 ```

```bash
```

docker swarm init
```bash
docker network create --driver=overlay ecosystem_network
```

```bash
docker stack deploy --prune --resolve-image always -c traefik.yml traefik
```

```bash
docker stack deploy --prune --resolve-image always -c portainer.yml portainer
```

Acesse URL de seu Site e Crie Usuario


</details>

Para instalar o Evolution, Vamos para aba Stacks e depois em add Stack

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/d1e54ab7-5c5f-4c28-902f-27266ed0abb7)

Colocamos o nome da stack (recomendo deixar evolution)


![image](https://github.com/cwmkt/dockerevolution/assets/91642837/ec07266c-d63b-48bc-a658-ccedb4fc3662)


#### Cole a stack do Evolution abaixo no seu portainer:

```bash
version: "3.7"
services:
  evolution_app:
    image: atendai/evolution-api:v1.7.1 # Altere para versão desejada
    command: ["node", "./dist/src/main.js"]
    volumes:
    - evolution_instances:/evolution/instances
    - evolution_store:/evolution/store
    - evolution_views:/evolution/views
    networks:
      - ecosystem_network
    environment:
    - SERVER_URL=https://api.seudominio.com.br 
    - DOCKER_ENV=true
    - LOG_LEVEL=ERROR
    - DEL_INSTANCE=false
    - CONFIG_SESSION_PHONE_CLIENT=NOME_SESSAO_APP
    - CONFIG_SESSION_PHONE_NAME=chrome
    - STORE_MESSAGES=true
    - STORE_MESSAGE_UP=true
    - STORE_CONTACTS=true
    - STORE_CHATS=true
    - CLEAN_STORE_CLEANING_INTERVAL=7200 # seconds === 2h
    - CLEAN_STORE_MESSAGES=true
    - CLEAN_STORE_MESSAGE_UP=true
    - CLEAN_STORE_CONTACTS=true
    - CLEAN_STORE_CHATS=true
    - AUTHENTICATION_TYPE=apikey
    - AUTHENTICATION_API_KEY=f4e9c72e34fd4485aa9e49295f9cbd8d
    - AUTHENTICATION_EXPOSE_IN_FETCH_INSTANCES=true
    - QRCODE_LIMIT=19021
    - QRCODE_COLOR=#81c244
    - WEBHOOK_GLOBAL_ENABLED=false
    - WEBHOOK_GLOBAL_URL=https://URL
    - WEBHOOK_GLOBAL_WEBHOOK_BY_EVENTS=false
    - WEBHOOK_EVENTS_APPLICATION_STARTUP=false
    - WEBHOOK_EVENTS_QRCODE_UPDATED=true
    - WEBHOOK_EVENTS_MESSAGES_SET=false
    - WEBHOOK_EVENTS_MESSAGES_UPSERT=true
    - WEBHOOK_EVENTS_MESSAGES_UPDATE=true
    - WEBHOOK_EVENTS_CONTACTS_SET=true
    - WEBHOOK_EVENTS_CONTACTS_UPSERT=true
    - WEBHOOK_EVENTS_CONTACTS_UPDATE=true
    - WEBHOOK_EVENTS_PRESENCE_UPDATE=true
    - WEBHOOK_EVENTS_CHATS_SET=true
    - WEBHOOK_EVENTS_CHATS_UPSERT=true
    - WEBHOOK_EVENTS_CHATS_UPDATE=true
    - WEBHOOK_EVENTS_CHATS_DELETE=true
    - WEBHOOK_EVENTS_GROUPS_UPSERT=true
    - WEBHOOK_EVENTS_GROUPS_UPDATE=true
    - WEBHOOK_EVENTS_GROUP_PARTICIPANTS_UPDATE=true
    - WEBHOOK_EVENTS_CONNECTION_UPDATE=true
    - REDIS_ENABLED=false
    - REDIS_URI=redis://redis:6379
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
        - node.role == manager
      labels:
      - traefik.enable=1
      - traefik.http.routers.evolution_app.rule=Host(`api.seudominio.com.br`)
      - traefik.http.routers.evolution_app.entrypoints=websecure
      - traefik.http.routers.evolution_app.priority=1
      - traefik.http.routers.evolution_app.tls.certresolver=letsencryptresolver
      - traefik.http.routers.evolution_app.service=evolution_app
      - traefik.http.services.evolution_app.loadbalancer.server.port=8080
      - traefik.http.services.evolution_app.loadbalancer.passHostHeader=1
volumes:
  evolution_instances:
    external: true
  evolution_store:
    external: true
  evolution_views:
    external: true

networks:
  ecosystem_network:
    external: true
```

### Obs: Lembre-se de alterar campos com as informações necessárias:

SERVER_URL: URL para API <br>
CONFIG_SESSION_PHONE_CLIENT: Nome da sessão no APP do Whatsapp <br>
traefik.http.routers.evolution_app.rule=Host(`api.seudominio.com.br`) <br>

Após toda alteração, podemos desativar o Enable access control e clicar em Deploy the stack

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/e39d7915-e223-4afe-98d0-fdc369138265)

**Pronto tudo Funcionando** ✅😎

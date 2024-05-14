<p align="center">
<img src="https://cwmkt.com.br/wp-content/uploads/2024/04/logo_github.png" width="240" />
<p align="center">Seja bem-vindo ao Guia de Instalação Docker Docuseal 🚀</p>
</p>
  
<p align="center">
<img src="https://whatsapp.com/favicon.ico" alt="WhatsAPP-logo" width="32" />
<span>Grupo WhatsaAPP: </span>
<a href="https://chat.whatsapp.com/K0HnkUZ41CYL8txpPWx2hO" target="_blank">Grupo</a>
</p>

<hr />
<hr />

Para instalar o Evolution, Vamos para aba Stacks e depois em add Stack

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/d1e54ab7-5c5f-4c28-902f-27266ed0abb7)

Colocamos o nome da stack (recomendo deixar evolution)


![image](https://github.com/cwmkt/dockerevolution/assets/91642837/ec07266c-d63b-48bc-a658-ccedb4fc3662)


#### Cole a stack do Evolution abaixo no seu portainer:

```bash
version: "3.8"

services:

  docuseal:
    image: docuseal/docuseal:latest
    ports:
      - "3000:3000"
    networks:
      - ecosystem_network
    volumes:
      - docuseal_volume:/data
    environment:
      POSTGRES_HOST: postgresql
      POSTGRES_USERNAME: user_name
      POSTGRES_PASSWORD: suasenhapostgres
      POSTGRES_DATABASE: seu_database
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=ecosystem_network"
        - "traefik.http.routers.docuseal.rule=Host(`dodocuseal.seudominio.com.br`)"
        - "traefik.http.routers.docuseal.tls=true"
        - "traefik.http.routers.docuseal.entrypoints=websecure"
        - "traefik.http.routers.docuseal.tls.certresolver=letsencryptresolver"
        - "traefik.http.routers.docuseal.service=docuseal"
        - "traefik.http.services.docuseal.loadbalancer.passHostHeader=true"
        - "traefik.http.services.docuseal.loadbalancer.server.port=3000"

volumes:
  docuseal_volume:
    external: true
    name: docuseal_volume

networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
```

### Obs: Lembre-se de alterar campos com as informações necessárias:

POSTGRES_USERNAME: user_name <br>
POSTGRES_PASSWORD: suasenhapostgres <br>
POSTGRES_DATABASE: seu_database <br>
- "traefik.http.routers.docuseal.rule=Host(`dodocuseal.seudominio.com.br`)" <br>

Após toda alteração, podemos desativar o Enable access control e clicar em Deploy the stack

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/e39d7915-e223-4afe-98d0-fdc369138265)

**Pronto tudo Funcionando** ✅😎

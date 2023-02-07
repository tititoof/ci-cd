# CI/CD
## Install Portainer on Cyric

### Update packages
```bash
sudo apt update
```

### Install prerequisite packages
```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

### Install Docker's offical GPG key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### Check fingerprint
```bash
sudo apt-key fingerprint 0EBFCD88
```

la réponse devrait être :
```bash
pub   rsa4096 2017-02-22 [SCEA]
     9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

### Ajout du dépôt Docker

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

### suppression des anciennes version de Docker

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### installation de la dernière version de docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### ajout du compte au groupe docker

```bash
sudo usermod -aG docker $USER
```

### redémarrage du serveur

```bash
sudo reboot
```

### initialisation du swarm
```bash
docker swarm init
```

réponse : 
```bash
Swarm initialized: current node (krw1rxgqmrf1423jvg3p1lpis) is now a manager.
 
To add a worker to this swarm, run the following command:
 
    docker swarm join --token SWMTKN-1-53c69qd36nlnsmj014k2eb5lyu1hpg7xmfuws1ojcnqgurwbgn-8v78bwtqgsh4zrpgenym1z8w8 192.168.1.204:2377
 
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### installation de docker-compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

réponse :

```bash
docker-compose version 1.24.0, build 0aa59064
```

### installation de portainer

fichier portainer.yml

```yml
version: '3.7'
services:
  portainer:
    image: portainer/portainer-ce:2.9.2
    restart: unless-stopped
    command: -H unix:///var/run/docker.sock
    ports:
      - 9000:9000
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./portainer/data:/data
    environment:
      TZ: "Europe/Paris"
```

créer le répertoire portainer
```bash
mkdir ./portainer
```

lancer le container avec la commande 
```bash
docker-compose -f ./portainer.yml up -d 
```

aller à l'adresse 
http://192.168.1.220:9000

Faire l'installation

### deploiement de l'agent portainer
```bash
curl -L https://downloads.portainer.io/agent-stack-ce211.yml -o agent-stack.yml && docker stack deploy --compose-file=agent-stack.yml portainer-agent
```

## installation postgresql sur ilmater
### Update packages
```bash
sudo apt update
```

### Install prerequisite packages
```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

### Install Docker's offical GPG key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### Check fingerprint
```bash
sudo apt-key fingerprint 0EBFCD88
```

la réponse devrait être :
```bash
pub   rsa4096 2017-02-22 [SCEA]
     9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

### Ajout du dépôt Docker

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

### suppression des anciennes version de Docker

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### installation de la dernière version de docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### ajout du compte au groupe docker

```bash
sudo usermod -aG docker $USER
```

### redémarrage du serveur

```bash
sudo reboot
```

### ajout du worker (noeud) au swarm 

```bash
docker swarm join --token SWMTKN-1-3pxmcepala1j3gwy5lcom3oru1icl382t4np5berrzd7wf12vm-70d398hfsq4tbn6zpyc4qlhu3 192.168.1.220:2377
```

### installation de docker-compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

réponse :

```bash
docker-compose version 1.24.0, build 0aa59064
```



### installation de postgresql

créer le répertoire ./postgresql/data
```bash
mkdir ./postgresql
mkdir ./postgresql/data
```

créer le fichier postgresql.yml
```yml
version: '3.7'

services:
  postgresql_db_1:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: toofytroll_dev_local
      POSTGRES_PASSWORD: uF8feX8VVrBkui
    logging:
      options:
        max-size: 10m
        max-file: "3"
    ports:
      - '5432:5432'
    volumes: 
      - ./postgresql/data:/var/lib/postgresql/data
    restart: always
```

lancer le container avec la commande 
```bash
docker-compose -f ./postgresql.yml up -d 
```

Se connecter à la base de données et lancer la création d'une base avec l'utilisateur

```sql
CREATE DATABASE gitea_dev;
CREATE USER gitea_dev_user WITH ENCRYPTED PASSWORD 'gitea_dev_pass';
GRANT ALL PRIVILEGES ON DATABASE gitea_dev TO gitea_dev_user;
```

#### installation de gitea sur cyric

se connecter sur cyric

créer les répertoires ./gitea/data
```bash
mkdir ./gitea
mkdir ./gitea/data
```

créer le fichier gitea.yml
```yml
version: "3"

services:
  server:
    image: gitea/gitea:latest
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=192.168.1.221:5432
      - GITEA__database__NAME=gitea_dev
      - GITEA__database__USER=gitea_dev_user
      - GITEA__database__PASSWD=gitea_dev_pass
    restart: always
    volumes:
      - ./gitea/data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
```

lancer le container avec la commande 
```bash
docker-compose -f ./gitea.yml up -d 
```

aller à l'url : http://192.168.1.220:3000/
faire l'installation

## installation de caddy
se connecter à cyric

créer les répertoires : 
./caddy/data
./caddy/config
et le fichier Caddyfile
./caddy/Caddyfile

```js
{
    # email to generate a valid SSL certificate
    email chartmann.35@gmail.com

    # HTTP/3 support
    servers {
        protocol {
            experimental_http3
        }
    }
}

gitea.chartman2.fr {
    reverse_proxy 192.168.1.220:3000
}

openproject.chartman2.fr {
    reverse_proxy 192.168.1.222:8080
}

jenkins.chartman2.fr {
    reverse_proxy 192.168.1.222:8081
}

sonar.chartman2.fr {
    reverse_proxy 192.168.1.223:9000
}

portainer.chartman2.fr {
    reverse_proxy 192.168.1.220:9000
}

dolibarr.chartman2.fr {
    reverse_proxy 192.168.1.220:8080
}

```

créer le fichier caddy.yml
```yml
version: '3'

services:
  caddy:
    image: caddy
    container_name: "caddy"
    volumes:
      - ./caddy/Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy/data:/data
      - ./caddy/config:/config
    ports:
      # HTTP
      - target: 80
        published: 80
        protocol: tcp
      # HTTPS
      - target: 443
        published: 443
        protocol: tcp
      # HTTP/3
      - target: 443
        published: 443
        protocol: udp
    restart: unless-stopped
```

lancer le container avec la commande 
```bash
docker-compose -f ./caddy.yml up -d 
```

## installation de jenkins sur bhaal

### Update packages
```bash
sudo apt update
```

### Install prerequisite packages
```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

### Install Docker's offical GPG key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### Check fingerprint
```bash
sudo apt-key fingerprint 0EBFCD88
```

la réponse devrait être :
```bash
pub   rsa4096 2017-02-22 [SCEA]
     9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

### Ajout du dépôt Docker

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

### suppression des anciennes version de Docker

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### installation de la dernière version de docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### ajout du compte au groupe docker

```bash
sudo usermod -aG docker $USER
```

### redémarrage du serveur

```bash
sudo reboot
```

### ajout du worker (noeud) au swarm 

```bash
docker swarm join --token SWMTKN-1-3pxmcepala1j3gwy5lcom3oru1icl382t4np5berrzd7wf12vm-70d398hfsq4tbn6zpyc4qlhu3 192.168.1.220:2377
```

### installation de docker-compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

réponse :

```bash
docker-compose version 1.24.0, build 0aa59064
```

## installation de jenkins

créer le répertoire ./jenkins/data
```bash
mkdir ./jenkins
mkdir ./jenkins/data
```

créer le fichier ./jenkins.yml
```yml
version: '3.7'

services:
  jenkins:
    image: jenkins/jenkins:lts
    privileged: true
    user: root
    ports:
      - 8081:8080
      - 50000:50000
    container_name: jenkins
    volumes:
      - ./jenkins/data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/local/bin/docker:/usr/local/bin/docker
```

lancer le service
```bash
docker-compose -f ./jenkins.yml up -d 
```

Faire l'installation

## installation des agents vuejs et rails sur bhaal

création des répertoires 
./rails_agent_jenkins/home
./rails_agent_jenkins/agent

```bash
mkdir ./rails_agent_jenkins
mkdir ./rails_agent_jenkins/{home,agent}
```

créer le fichier rails_agent_jenkins.yml
```yml
version: '3.7'

services:
  jenkins_agent_rails:
    image: ghcr.io/tititoof/jenkins-agent-rails:latest
    ports:
      - 8082:8080
      - 50001:50000
    container_name: jenkins_agent_rails
    command: -url http://192.168.1.222:8081 c607358ac1f58668e7d752eb7c17e01ad00abbed01064ca98b3fad1d9f0c5f70 agent_rails_1
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - ./rails_agent_jenkins/home:/var/jenkins_home
      - ./rails_agent_jenkins/agent:/home/jenkins/agent
```

lancer le service
```bash
docker-compose -f ./rails_agent_jenkins.yml up -d 
```

création des répertoires 
./vuejs_agent_jenkins/home
./vuejs_agent_jenkins/agent

```bash
mkdir ./vuejs_agent_jenkins
mkdir ./vuejs_agent_jenkins/{home,agent}
```

créer le fichier vuejs_agent_jenkins.yml
```yml
version: '3.7'

services:
  jenkins_agent_vuejs:
    image: ghcr.io/tititoof/jenkins-agent-vuejs
    ports:
      - 8083:8080
      - 50002:50000
    container_name: jenkins_agent_vuejs
    command: -url http://192.168.1.222:8081 14fddb0ccd3827b479bd308f1cf0c540960d706350e985c8c90c4b7797522c16 agent_vuejs_1
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - ./vuejs_agent_jenkins/home:/var/jenkins_home
      - ./vuejs_agent_jenkins/agent:/home/jenkins/agent
```

lancer le service
```bash
docker-compose -f ./vuejs_agent_jenkins.yml up -d 
```

## installation de sonarqube et openproject sur baine

### Update packages
```bash
sudo apt update
```

### Install prerequisite packages
```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

### Install Docker's offical GPG key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### Check fingerprint
```bash
sudo apt-key fingerprint 0EBFCD88
```

la réponse devrait être :
```bash
pub   rsa4096 2017-02-22 [SCEA]
     9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

### Ajout du dépôt Docker

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

### suppression des anciennes version de Docker

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### installation de la dernière version de docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### ajout du compte au groupe docker

```bash
sudo usermod -aG docker $USER
```

### redémarrage du serveur

```bash
sudo reboot
```

### ajout du worker (noeud) au swarm 

```bash
docker swarm join --token SWMTKN-1-3pxmcepala1j3gwy5lcom3oru1icl382t4np5berrzd7wf12vm-70d398hfsq4tbn6zpyc4qlhu3 192.168.1.220:2377
```

### installation de docker-compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

réponse :

```bash
docker-compose version 1.24.0, build 0aa59064
```

### installation de sonarqube
créer les répertoires : 
./sonarqube
./sonarqube/{conf,extensions,logs,data}

```bash
mkdir ./sonarqube
mkdir ./sonarqube/{conf,extensions,logs,data}
```

créer le fichier sonarqube.yml
```yml
version: "3.7"

services:
  sonarqube:
    image: sonarqube:lts
    ports:
      - "9000:9000"
    volumes:
      - "./sonarqube/conf:/opt/sonarqube/conf"
      - "./sonarqube/extensions:/opt/sonarqube/extensions"
      - "./sonarqube/logs:/opt/sonarqube/logs"
      - "./sonarqube/data:/opt/sonarqube/data"
    environment:
      - sonar.jdbc.username=sonarqube_dev_user
      - sonar.jdbc.password=sonarqube_dev_pass
      - sonar.jdbc.url=jdbc:postgresql://192.168.1.221:5432/sonarqube_dev
    ulimits:
      nproc: 131072
      nofile:
        soft: 8192
        hard: 131072
```

lancer le service
```bash
docker-compose -f ./sonarqube.yml up -d 
```

### installation d'openproject
créer les répertoires : 
./openproject
./openproject/{pgdata,logs,static,bundle,tmp}

```bash
mkdir ./openproject
mkdir ./openproject/{pgdata,logs,static,bundle,tmp}
```

créer le fichier openproject.yml
```yml
version: "3.7"

services:
  openproject:
    image: openproject/community:12
    environment:
      - "DATABASE_URL=postgresql://open_project_dev_user:open_project_dev_pass@192.168.1.221/open_project_dev"
      - "USE_PUMA=true"
      # set to true to enable the email receiving feature. See ./docker/cron for more options
      - "IMAP_ENABLED=false"
      - USER_UID=1000
      - USER_GID=1000
    volumes:
      - ./openproject/logs:/var/log/supervisor
      - ./openproject/static:/var/openproject/assets
      - ./openproject/tmp:/home/dev/openproject/tmp
    expose:
      - "8080"
    ports:
      - "8080:8080"
```

lancer le service
```bash
docker-compose -f ./openproject.yml up -d 
```
# DevOps — Docker + Jenkins

---

## DOCKER

```bash
docker ps          # containers rodando
docker images      # imagens baixadas
```

### Imagens essenciais

```bash
docker pull nginx:alpine
docker pull jenkins/jenkins:lts-jdk17
```

### Comandos 

```bash
docker ps                 
docker ps -a               
docker logs -f NOME       
docker stop NOME
docker start NOME
docker restart NOME
docker rm -f NOME
docker exec -it NOME bash 
docker system prune        # limpa recursos não usados
```

### Subir o site deste projeto

```bash
docker build -t meu-site .
docker run -d --name meu-site -p 80:80 meu-site
```

Para conectar o container a pastas/portas do seu PC, usa-se `-v` (volume) e `-p` (porta):

```bash
docker run -d -v minha_pasta:/app -p 8080:80 imagem
```

---

## JENKINS

### Subir o Jenkins

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk17
```

- **8080** = painel web · **50000** = agentes de build
- **jenkins_home** = volume que guarda tudo (não apague!)

A primeira subida leva 1 a 3 min (acompanhe: `docker logs -f jenkins`).

### Acessar e configurar

1. Abra `http://localhost:8080`
2. Peça a senha inicial:

   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```

3. Cole a senha → **Continue** → instale os plugins sugeridos.
4. Crie o usuário admin (ou clique em **"Skip and continue as admin"**: usuário `admin` + senha inicial).
5. Configure a URL (mantenha `http://localhost:8080`) → finalize.

> Senha atual desta instalação: `0b34c1ebf23f4861bf5a11600d3a84b3` (muda ao criar o usuário).

### Pipeline de build (Jenkinsfile)

Crie um job do tipo **Pipeline** apontando para o `Jenkinsfile` do repositório.

Ele segue o padrão: `Checkout → Build (docker) → Test → Package (push) → Deploy`.

Exemplo mínimo dessa estrutura:

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Build')    { steps { sh 'docker build -t meu-site .' } }
        stage('Deploy')   { steps { sh 'docker run -d -p 80:80 meu-site' } }
    }
}
```

### Para o Jenkins conseguir rodar docker

O Jenkins precisa enxergar o Docker da máquina:

```bash
docker run -d --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk17
```

Se der erro de permissão, rode com `-u root`.

### Comandos

```bash
docker ps --filter name=jenkins                  # está rodando?
docker logs -f jenkins                           # logs/inicialização
docker stop jenkins && docker rm jenkins         # remover (mantém dados)
docker exec -it jenkins bash                     # terminal interno
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword  # senha inicial
```

### Portas e agentes

- **50000** = agentes que conectam no master (Dashboard , Manage Jenkins , Nodes).
- Trocar a porta do painel: use outro `-p`, ex.: `-p 9000:8080`.

---


```bash
docker ps
docker logs -f jenkins                      # ver Jenkins
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
docker stop jenkins && docker start jenkins # reiniciar
docker build -t meu-site . && docker run -d -p 80:80 meu-site   # publicar o site
```

**Painel Jenkins:** http://localhost:8080 · **Site:** http://localhost:80
# n8n_nginx_docker

Ferramentas envolvidas:

1. n8n - Automação de workflows rodando com postgres
2. nginx - Proxy reverso para não expor o n8n diretamente e adicionar certificado SSL

OBS: Rodando em uma VM linux com a instalação prévia do docker

## Instalação do Docker

### 1. Preparação da VPS

#### 1.1 Atualizar o sistema

```sudo apt update && sudo apt upgrade -y```

#### 1.2 Instalar Docker

```
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```
Saia da sessão SSH e entre novamente para aplicar o grupo docker.

#### 1.3 Instalar Docker Compose (plugin oficial)

```sudo apt install docker-compose-plugin -y```

Verifique:

```docker compose version```

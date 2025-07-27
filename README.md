# 🚀 Instalação do Docker + Docker Compose v2 no Ubuntu 24.04

Este repositório contém um script simples e confiável para instalar o **Docker Engine** e o **Docker Compose v2** no Ubuntu 24.04 LTS.

## 📦 O que o script faz

- Atualiza os pacotes do sistema
- Instala dependências básicas
- Adiciona a chave GPG e o repositório oficial do Docker
- Instala:
  - Docker Engine
  - Docker CLI
  - Docker Compose v2 (como plugin)
- Adiciona o usuário atual ao grupo `docker`
- Verifica se as instalações foram concluídas com sucesso

## 📁 Script

  ```
#!/bin/bash

set -e

echo "🔧 Atualizando pacotes..."
sudo apt update
sudo apt upgrade -y

echo "📦 Instalando dependências..."
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

echo "📂 Criando diretório para chave GPG do Docker..."
sudo install -m 0755 -d /etc/apt/keyrings

echo "🔑 Adicionando chave GPG oficial do Docker..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "🔒 Definindo permissões para a chave GPG..."
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "📝 Adicionando repositório Docker..."
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "🔄 Atualizando pacotes após adicionar repositório Docker..."
sudo apt update

echo "🐳 Instalando Docker Engine e Compose v2..."
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "🧪 Verificando Docker..."
sudo docker --version

echo "🧪 Verificando Docker Compose v2..."
sudo docker compose version

echo "👤 Adicionando usuário atual ao grupo docker..."
sudo usermod -aG docker $USER

echo "✅ Instalação concluída com sucesso!"
echo "⚠️ Saia e entre novamente na sessão para usar o Docker sem sudo."

```

## 📋 Pré-requisitos

- Distribuição baseada em **Ubuntu 24.04 LTS**
- Permissões de **sudo**

## Dê permissão de execução ao script
```
chmod +x install-docker.sh
```

## Execute o script
```
./install-docker.sh
```

## Após a instalação, reinicie a sessão (logout/login) para aplicar o grupo docker

✅ Verificações
Verifique se o Docker e o Compose v2 foram instalados corretamente
```
docker --version
docker compose version
```

🐳 Exemplo de uso com Compose
Crie um arquivo docker-compose.yml e use o comando
```
docker compose up -d
```

🛠️ Testado em
```
Ubuntu 24.04 LTS
Docker CE v24+
Docker Compose v2 (plugin oficial)
```


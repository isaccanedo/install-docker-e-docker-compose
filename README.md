# 🚀  Instalação do Docker + Docker Compose + Java 21 em uma instância VM Ubuntu 24.04 no Amazon EC2

Este repositório contém um script simples e confiável para instalar o **Docker Engine** e o **Docker Compose v2** e o **Java 21** no Ubuntu 24.04 LTS.

## 📦 O que o script faz

- Atualiza os pacotes do sistema
- Instala dependências básicas
- Adiciona a chave GPG e o repositório oficial do Docker
- Instala:
  - Java 21 (OpenJDK)
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
apt update
apt upgrade -y

echo "📦 Instalando dependências..."
apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release \
  wget

echo "☕ Instalando Java 21 (OpenJDK)..."
mkdir -p /etc/apt/keyrings
wget -O- https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --dearmor -o /etc/apt/keyrings/adoptium.gpg

echo "deb [signed-by=/etc/apt/keyrings/adoptium.gpg] https://packages.adoptium.net/artifactory/deb $(lsb_release -cs) main" | \
  tee /etc/apt/sources.list.d/adoptium.list > /dev/null

apt update
apt install -y temurin-21-jdk

echo "🧪 Verificando Java..."
java -version

echo "📦 Instalando Apache Maven..."
apt install -y maven

echo "🧪 Verificando Maven..."
mvn -version

echo "📂 Criando diretório para chave GPG do Docker..."
install -m 0755 -d /etc/apt/keyrings

echo "🔑 Adicionando chave GPG oficial do Docker..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "🔒 Definindo permissões para a chave GPG..."
chmod a+r /etc/apt/keyrings/docker.gpg

echo "📝 Adicionando repositório Docker..."
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "🔄 Atualizando pacotes após adicionar repositório Docker..."
apt update

echo "🐳 Instalando Docker Engine e Compose v2..."
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "🧪 Verificando Docker..."
docker --version

echo "🧪 Verificando Docker Compose v2..."
docker compose version

echo "👤 Adicionando usuário padrão ao grupo docker..."
usermod -aG docker ubuntu  # Modifique se o usuário for outro

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
Verifique se o Docker, Docker Compose v2 e o Java 21 foram instalados corretamente
```
java --version
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

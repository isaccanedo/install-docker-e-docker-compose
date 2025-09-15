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

# Log de execução
exec > >(tee /var/log/user-data.log) 2>&1

echo "🚀 Iniciando configuração da instância EC2..."

# Verificar se está executando como root
if [ "$EUID" -ne 0 ]; then 
    echo "❌ Este script deve ser executado como root ou com sudo"
    exit 1
fi

# Aguardar outros processos apt terminarem
echo "⏳ Verificando processos apt..."
while fuser /var/lib/dpkg/lock-frontend >/dev/null 2>&1; do
    echo "⏳ Aguardando outros processos apt terminarem..."
    sleep 5
done

# Configurar não-interativo para evitar prompts
export DEBIAN_FRONTEND=noninteractive

echo "🔧 Atualizando pacotes..."
apt update
apt upgrade -y

echo "📦 Instalando dependências..."
apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    wget \
    git \
    maven \
    gnupg2 \
    software-properties-common \
    apt-transport-https

echo "☕ Instalando Java 21 (OpenJDK)..."
mkdir -p /etc/apt/keyrings
wget -O- https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --dearmor -o /etc/apt/keyrings/adoptium.gpg
echo "deb [signed-by=/etc/apt/keyrings/adoptium.gpg] https://packages.adoptium.net/artifactory/deb $(lsb_release -cs) main" | \
    tee /etc/apt/sources.list.d/adoptium.list
apt update
apt install -y temurin-21-jdk

echo "🧪 Verificando Java..."
java -version

echo "🧪 Verificando Git..."
git --version

echo "🧪 Verificando Maven..."
mvn -version

echo "📂 Criando diretório para chave GPG do Docker..."
install -m 0755 -d /etc/apt/keyrings

echo "🔑 Adicionando chave GPG oficial do Docker..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

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

echo "▶️ Iniciando e habilitando Docker..."
systemctl start docker
systemctl enable docker

echo "👤 Adicionando usuário ubuntu ao grupo docker..."
usermod -aG docker ubuntu

# Criar script para aplicar grupo docker na próxima sessão
echo "🔧 Criando script de configuração pós-boot..."
cat > /home/ubuntu/apply-docker-group.sh << 'EOF'
#!/bin/bash
echo "🐳 Aplicando grupo docker para a sessão atual..."
newgrp docker << 'DOCKERGRP'
echo "✅ Grupo docker aplicado! Testando Docker..."
docker --version
docker run --rm hello-world
echo "🎉 Docker está funcionando!"
exit
DOCKERGRP
EOF

chmod +x /home/ubuntu/apply-docker-group.sh
chown ubuntu:ubuntu /home/ubuntu/apply-docker-group.sh

# Criar alias para facilitar o uso
echo "alias docker-setup='/home/ubuntu/apply-docker-group.sh'" >> /home/ubuntu/.bashrc

# Configurar variáveis de ambiente Java
echo "🌍 Configurando variáveis de ambiente Java..."
echo 'export JAVA_HOME=/usr/lib/jvm/temurin-21-jdk-amd64' >> /etc/environment
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /etc/environment

# Aplicar as variáveis para o usuário ubuntu
echo 'export JAVA_HOME=/usr/lib/jvm/temurin-21-jdk-amd64' >> /home/ubuntu/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /home/ubuntu/.bashrc

# Corrigir permissões do .bashrc
chown ubuntu:ubuntu /home/ubuntu/.bashrc

echo "✅ Instalação concluída com sucesso!"
echo "📝 Log salvo em: /var/log/user-data.log"
echo ""
echo "🐳 PARA USAR DOCKER IMEDIATAMENTE:"
echo "   Execute: ./apply-docker-group.sh"
echo "   Ou use: docker-setup"
echo ""
echo "⚠️ ALTERNATIVA: Faça logout/login para usar Docker normalmente"
echo "🎉 Instância pronta para uso!"

# Criar arquivo de status para indicar conclusão
touch /tmp/setup-complete
echo "$(date): Setup completed successfully" > /tmp/setup-complete
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

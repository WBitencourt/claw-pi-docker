

# 🤖 claw-pi-docker

Servidor OpenClaw em Docker para o sistema operacional Raspberry Pi.

## Guia de configuração

Siga as etapas abaixo na ordem. Cada uma tem um guia detalhado — clique no link para abrir o README correspondente.

- [1. Raspberry Pi OS](#1-raspberry-pi-os) — preparar o sistema no Raspberry (imagem do OS, SSH, rede, etc.).
- [2. Docker](#2-docker) — instalar e configurar o Docker no Pi para rodar containers.
- [3. Open-Claw](#3-open-claw) — configurar e rodar o servidor OpenClaw em container.

---

### 1. Raspberry Pi OS

Base do servidor: gravar a imagem do Raspberry Pi OS no cartão SD, configurar usuário, SSH e rede. Sem esta etapa o Pi não estará pronto para os próximos passos.

**Guia completo:** [config/pi-OS/README.md](config/pi-OS/README.md)

### 2. Docker

Instalação do Docker no Raspberry Pi (método rápido ou repositório oficial). Necessário para executar o OpenClaw em container.

**Guia completo:** [config/docker/README.md](config/docker/README.md)

### 3. Open-Claw

Criação do projeto com Dockerfile e docker-compose, build do container, onboarding e acesso ao dashboard do OpenClaw.

**Guia completo:** [config/open-claw/README.md](config/open-claw/README.md)

---

## Comandos úteis

Comandos úteis para monitorar o container e o servidor:

| O que fazer | Comando |
|-------------|---------|
| Ver uso de CPU/memória do container Open-Claw | `docker stats open-claw` |
| Ver processos do servidor (interativo) | `htop` |
| Ver temperatura da CPU do servidor (Raspberry Pi) | `vcgencmd measure_temp` |

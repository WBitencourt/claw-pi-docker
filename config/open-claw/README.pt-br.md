# 🦞 Configurando o Open-Claw

Este método requer um servidor com Docker instalado. Se ainda não configurou o Docker no seu Raspberry Pi, siga nosso guia em [Docker Setup Guide](../docker/README.md).

## 📂 1. Preparação do Diretório

Primeiro, criamos uma pasta dedicada para manter os arquivos de configuração e persistência organizados:

```bash
mkdir ~/openclaw && cd ~/openclaw
```

## 🐳 2. Criando o Dockerfile (Hardened Multi-stage)

Utilizamos um build de múltiplos estágios para garantir uma imagem leve e segura, rodando com um usuário não-root (UID 1001).

Crie o arquivo na pasta do projeto (ex.: `~/openclaw`) ou use o conteúdo do repositório: [Dockerfile](Dockerfile).

## ⚙️ 3. Configurando o Docker Compose

O Compose gerencia o container e limita os recursos para não sobrecarregar o hardware do Raspberry Pi.

Crie o arquivo na pasta do projeto ou use o conteúdo do repositório: [docker-compose.yml](docker-compose.yml).

## 🛡️ 4. Permissões de Host e Firewall

Para que o container consiga escrever na pasta do Raspberry Pi:

```bash
# 1. Garante que o host respeite o UID 1001 do container
sudo chown -R 1001:1001 ~/openclaw/volume
```

Confira o estado atual do seu firewall com: 

```bash
# Mostra o status sem numeracao
sudo ufw status

# Mostra o status com numeracao, facilita quando precisar deletar uma regra conforme proximo comando
sudo ufw status numbered

# Facilita quando precisar deletar uma regra
sudo ufw delete 1
```



Se estiver seguindo esse tutorial sem usar a rede mesh possivelmente esse comando ira retornar algo como:


| To          | Action | From          |
| ----------- | ------ | ------------- |
| 22/tcp      | ALLOW  | Anywhere      |
| 22/tcp (v6) | ALLOW  | Anywhere (v6) |


Isso indica que temos acesso via SSH na porta 22/tcp de qualquer lugar, porem sem acesso a porta 18789 por exemplo, precisamos liberar essa porta para acessarmos o dashboard do open-claw futuramente.

```bash
# Permite acesso pela rede local
sudo ufw allow from 192.168.10.0/24 to any port 18789
```

Se estiver seguindo os meus passos com a rede meshnet da NordVPN, teremos algo como:


| To                        | Action | From            |
| ------------------------- | ------ | --------------- |
| Anywhere on nordlynx      | ALLOW  | Anywhere        |
| Anywhere (v6) on nordlynx | ALLOW  | Anywhere (v6)   |
| 22/tcp                    | ALLOW  | 192.168.10.0/24 |


Nesse caso minha rede nordlynx esta configurada com o anywhere on nordlynx, nada precisa ser alterado, nao preciso liberar a porta 18789, pois qualquer dispositivo meu conectado nessa rede privativa da nordvpn pode se comunicar com qualquer porta do meu servidor. Porém se quiser ser mais restritivo, poderiamos deixar apenas acesso apenas a porta 22 com protocolo tcp e a porta 18789 ambos na rede meshnet da NordVPN.

```bash
# 1. Remove todas as permissoes "anywhere on nordlynx" da rede meshnet (NordVPN)
sudo ufw delete allow in on nordlynx

# 2. Permite acesso via SSH na rede meshnet (NordVPN)
sudo ufw allow in on nordlynx to any port 22 proto tcp

#3. Libera a porta 189789 na rede meshnet (NordVPN)
sudo ufw allow in on nordlynx to any port 18789
```

OBS: Quando essas regras são necessárias: As regras são importantes quando a interface NordLynx não está configurada para aceitar conexões em qualquer porta ("Anywhere on nordlynx")..

Ao executar os comandos teríamos algo assim:


| To                      | Action | From          |
| ----------------------- | ------ | ------------- |
| 22/tcp on nordlynx      | ALLOW  | Anywhere      |
| 18789 on nordlynx       | ALLOW  | Anywhere      |
| 22/tcp (v6) on nordlynx | ALLOW  | Anywhere (v6) |
| 18789 (v6) on nordlynx  | ALLOW  | Anywhere (v6) |


Poderiamos restringir tambem de qual IP podemos receber conexões, apenas da nossa rede local, ou ate mesmo de um IP fixo alterando o from no firewall.

## 🚀 5. Build e Inicialização

Certifique-se de estar no diretório onde estão o `Dockerfile` e o `docker-compose.yml` (ex.: `~/openclaw`). Se necessário, faça login no Docker Hub e suba o serviço:

```bash
cd ~/openclaw

# Login no registro (se necessário)
docker login

# Força o build e start do container
docker compose up --build -d

# Acompanhar o consumo de recursos
docker stats
```

## 🔑 6. Onboarding (Configuração Inicial)

Com o container rodando, execute o assistente para configurar suas chaves de API e senha do Dashboard:

```bash
docker exec -it open-claw openclaw onboard
```

ou, acesse diretamente o terminal do container, caso escolha essa alternativa os demais comando nesse passo a passo explicado, estando dentro do terminal do container, ignorar `docker exec -it open-claw` e executar diretamente, por exemplo `openclaw --version` 

```bash
docker exec -it open-claw /bin/bash
openclaw onboard
```

Após configurar, você poderá acessar o dashboard pelo navegador (por exemplo, no Galaxy Book: `http://IP_RASPBERRY:18789` ou via uma das opções de acesso descritas mais abaixo).

Exemplo de saída do assistente (valores sensíveis substituídos por placeholders neste guia):

```
🦞 OpenClaw 2026.3.11 (29dc654)
   Pairing codes exist because even bots believe in consent—and good security hygiene.

▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
██░▄▄▄░██░▄▄░██░▄▄▄██░▀██░██░▄▄▀██░████░▄▄▀██░███░██
██░███░██░▀▀░██░▄▄▄██░█░█░██░█████░████░▀▀░██░█░█░██
██░▀▀▀░██░█████░▀▀▀██░██▄░██░▀▀▄██░▀▀░█░██░██▄▀▄▀▄██
▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀
                  🦞 OPENCLAW 🦞                    
 
┌  OpenClaw onboarding
│
◇  Security ─────────────────────────────────────────────────────────────────────────────────╮
│                                                                                            │
│  Security warning — please read.                                                           │
│                                                                                            │
│  OpenClaw is a hobby project and still in beta. Expect sharp edges.                        │
│  By default, OpenClaw is a personal agent: one trusted operator boundary.                  │
│  This bot can read files and run actions if tools are enabled.                             │
│  A bad prompt can trick it into doing unsafe things.                                       │
│                                                                                            │
│  OpenClaw is not a hostile multi-tenant boundary by default.                               │
│  If multiple users can message one tool-enabled agent, they share that delegated tool      │
│  authority.                                                                                │
│                                                                                            │
│  If you’re not comfortable with security hardening and access control, don’t run           │
│  OpenClaw.                                                                                 │
│  Ask someone experienced to help before enabling tools or exposing it to the internet.     │
│                                                                                            │
│  Recommended baseline:                                                                     │
│  - Pairing/allowlists + mention gating.                                                    │
│  - Multi-user/shared inbox: split trust boundaries (separate gateway/credentials, ideally  │
│    separate OS users/hosts).                                                               │
│  - Sandbox + least-privilege tools.                                                        │
│  - Shared inboxes: isolate DM sessions (`session.dmScope: per-channel-peer`) and keep      │
│    tool access minimal.                                                                    │
│  - Keep secrets out of the agent’s reachable filesystem.                                   │
│  - Use the strongest available model for any bot with tools or untrusted inboxes.          │
│                                                                                            │
│  Run regularly:                                                                            │
│  openclaw security audit --deep                                                            │
│  openclaw security audit --fix                                                             │
│                                                                                            │
│  Must read: https://docs.openclaw.ai/gateway/security                                      │
│                                                                                            │
├────────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  I understand this is personal-by-default and shared/multi-user use requires lock-down. Continue?
│  Yes
│
◇  Onboarding mode
│  Manual
│
◇  Existing config detected ──╮
│                             │
│  No key settings detected.  │
│                             │
├─────────────────────────────╯
│
◇  Config handling
│  Update values
│
◇  What do you want to set up?
│  Local gateway (this machine)
│
◇  Workspace directory
│  /app/data/.openclaw/workspace
│
◇  Model/auth provider
│  OpenRouter
│
◇  How do you want to provide this API key?
│  Paste API key now
│
◇  Enter OpenRouter API key
│  sk-or-v1-[REDACTED]  (substitua pela sua chave em https://openrouter.ai; nunca publique chaves reais)
│
◇  Model configured ─────────────────────╮
│                                        │
│  Default model set to openrouter/auto  │
│                                        │
├────────────────────────────────────────╯
│
◇  Default model
│  openrouter/google/gemma-3-27b-it:free
│
◇  Gateway port
│  18789
│
◇  Gateway bind
│  LAN (0.0.0.0)
│
◇  Gateway auth
│  Password
│
◇  Tailscale exposure
│  Off
│
◇  How do you want to provide the gateway password?
│  Enter password now
│
◇  Gateway password
│  [SENHA_DO_GATEWAY]  (defina uma senha forte; este é apenas um exemplo)
│
◇  Channel status ────────────────────────────╮
│                                             │
│  Telegram: needs token                      │
│  WhatsApp (default): not linked             │
│  Discord: needs token                       │
│  Slack: needs tokens                        │
│  Signal: needs setup                        │
│  signal-cli: missing (signal-cli)           │
│  iMessage: needs setup                      │
│  imsg: missing (imsg)                       │
│  IRC: not configured                        │
│  Google Chat: not configured                │
│  LINE: not configured                       │
│  Feishu: install plugin to enable           │
│  Google Chat: install plugin to enable      │
│  Nostr: install plugin to enable            │
│  Microsoft Teams: install plugin to enable  │
│  Mattermost: install plugin to enable       │
│  Nextcloud Talk: install plugin to enable   │
│  Matrix: install plugin to enable           │
│  BlueBubbles: install plugin to enable      │
│  LINE: install plugin to enable             │
│  Zalo: install plugin to enable             │
│  Zalo Personal: install plugin to enable    │
│  Synology Chat: install plugin to enable    │
│  Tlon: install plugin to enable             │
│                                             │
├─────────────────────────────────────────────╯
│
◇  Configure chat channels now?
│  No
Config overwrite: /app/data/config.toml (sha256 [REDACTED] -> [REDACTED], backup=/app/data/config.toml.bak)
Updated $OPENCLAW_HOME/config.toml
Workspace OK: $OPENCLAW_HOME/.openclaw/workspace
Sessions OK: $OPENCLAW_HOME/state/agents/main/sessions
│
◇  Web search ────────────────────────────────────────╮
│                                                     │
│  Web search lets your agent look things up online.  │
│  Choose a provider and paste your API key.          │
│  Docs: https://docs.openclaw.ai/tools/web           │
│                                                     │
├─────────────────────────────────────────────────────╯
│
◆  Search provider
│  ○ Brave Search (Structured results · country/language/time filters)
│  ○ Gemini (Google Search)
│  ○ Grok (xAI)
│  ○ Kimi (Moonshot)
│  ○ Perplexity Search
│  ● Skip for now
└
◇  Skills status ─────────────╮
│                             │
│  Eligible: 2                │
│  Missing requirements: 42   │
│  Unsupported on this OS: 7  │
│  Blocked by allowlist: 0    │
│                             │
├─────────────────────────────╯
│
◇  Configure skills now? (recommended)
│  No
│
◇  Hooks ──────────────────────────────────────────────────────────────────╮
│                                                                          │
│  Hooks let you automate actions when agent commands are issued.          │
│  Example: Save session context to memory when you issue /new or /reset.  │
│                                                                          │
│  Learn more: https://docs.openclaw.ai/automation/hooks                   │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────╯
│
◇  Enable hooks?
│  Skip for now
Config overwrite: /app/data/config.toml (sha256 [REDACTED] -> [REDACTED], backup=/app/data/config.toml.bak)
│
◇  Systemd ───────────────────────────────────────────────────────────────────────────────╮
│                                                                                         │
│  Systemd user services are unavailable. Skipping lingering checks and service install.  │
│                                                                                         │
├─────────────────────────────────────────────────────────────────────────────────────────╯

│

◇  
Agents: main (default)
Heartbeat interval: 30m (main)
Session store (main): /app/data/state/agents/main/sessions/sessions.json (1 entries)
- agent:main:main (8293m ago)
│
◇  Optional apps ────────────────────────╮
│                                        │
│  Add nodes for extra features:         │
│  - macOS app (system + notifications)  │
│  - iOS app (camera/canvas)             │
│  - Android app (camera/canvas)         │
│                                        │
├────────────────────────────────────────╯
│
◇  Control UI ──────────────────────────────────────────────────────────────────────╮
│                                                                                   │
│  Web UI: http://172.18.0.2:18789/                                                 │
│  Gateway WS: ws://172.18.0.2:18789                                                │
│  Gateway: not detected (SECURITY ERROR: Gateway URL "ws://172.18.0.2:18789" uses  │
│  plaintext ws:// to a non-loopback address.)                                      │
│  Docs: https://docs.openclaw.ai/web/control-ui                                    │
│                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────╯
│
◇  Workspace backup ────────────────────────────────────────╮
│                                                           │
│  Back up your agent workspace.                            │
│  Docs: https://docs.openclaw.ai/concepts/agent-workspace  │
│                                                           │
├───────────────────────────────────────────────────────────╯
│
◇  Security ──────────────────────────────────────────────────────╮
│                                                                 │
│  Running agents on your computer is risky — harden your setup:  │
│  https://docs.openclaw.ai/security                              │
│                                                                 │
├─────────────────────────────────────────────────────────────────╯
│
◇  Enable zsh shell completion for openclaw?
│  Yes
Failed to install completion: Error: EACCES: permission denied, mkdir '/nonexistent'
│
◇  Shell completion ───────────────────────────────────────────────────────╮
│                                                                          │
│  Shell completion installed. Restart your shell or run: source ~/.zshrc  │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────╯
│
◇  Web search ───────────────────────────────────────╮
│                                                    │
│  Web search was skipped. You can enable it later:  │
│    openclaw configure --section web                │
│                                                    │
│  Docs: https://docs.openclaw.ai/tools/web          │
│                                                    │
├────────────────────────────────────────────────────╯
│
◇  What now ─────────────────────────────────────────────────────────────╮
│                                                                        │
│  What now: https://openclaw.ai/showcase ("What People Are Building").  │
│                                                                        │
├────────────────────────────────────────────────────────────────────────╯
│
└  Onboarding complete. Use the dashboard link above to control OpenClaw.

```

**Dica — modelos gratuitos vs. crédito pago:** Os modelos marcados como *free* no OpenRouter podem falhar ou não responder com frequência. Após liberar um limite de uso pago na sua conta OpenRouter (ex.: um valor mínimo em dólares), as chamadas aos modelos tendem a funcionar de forma estável. Para uso confiável, considere configurar um crédito mínimo na [OpenRouter](https://openrouter.ai).

### Endereços confiáveis (CORS)

Adicione os endereços de onde você vai acessar o dashboard para evitar erro de CORS. Ajuste a lista abaixo conforme seu uso:

- **IP local do Raspberry:** exige que o servidor tenha HTTPS configurado.
- **Nome do servidor (hostname):** exige que o servidor tenha HTTPS configurado.
- **Nome do servidor na meshnet:** exige que o servidor tenha HTTPS configurado.

Exemplo (substitua os IPs e hostnames pelos seus — ex.: IP da LAN, IP da meshnet, nomes locais):

```bash
docker exec -it open-claw openclaw config set gateway.controlUi.allowedOrigins '["http://localhost:18789","https://IP_DA_SUA_LAN:18789","https://IP_MESHNET:18789","https://raspberry-server.local:18789","https://raspberry-server.nord:18789"]'
docker compose restart open-claw
```

## Opções de acesso ao dashboard

O dashboard permite, em resumo, **duas formas de acesso:** (1) **localhost** — como o servidor não tem interface gráfica, usou-se túnel SSH do cliente (ex.: Galaxy Book) ao servidor, de modo que o OpenClaw “vê” a conexão como local (Opção 1 abaixo); (2) **acesso direto por URL** — exige certificado HTTPS no servidor (reverse proxy, ex.: Nginx, ou Cloudflare Tunnel que pode fornecer isso; em rede privada/mesh também é possível usar HTTPS com Nginx para o “cadeado verde” e Secure Context). Este guia foi escrito usando a **Opção 1**; outras opções serão documentadas conforme testadas.

### Visão geral: acesso público vs privado

**1. Acesso público (qualquer lugar do mundo, sem VPN)**  
O serviço fica exposto na internet.

- **Cloudflare Tunnel:** Não é preciso abrir portas no roteador. O Raspberry “disca” para a Cloudflare e cria um túnel. Acesso via algo como `openclaw.seudominio.com`. Opção mais segura para acesso público.
- **Port forwarding (redirecionamento de portas):** Abrir a porta 443 no roteador apontando para o IP do Raspberry. Exige DNS dinâmico (DDNS) se não houver IP fixo e um reverse proxy (ex.: Nginx) para HTTPS.

**2. Acesso privado / rede mesh (somente seus dispositivos)**  
O serviço não fica na internet pública; só existe na “nuvem privada” que você criou.

- **Tailscale / NordVPN Meshnet:** O Raspberry e o cliente (ex.: Galaxy Book) formam uma rede virtual segura. De qualquer lugar, com a mesh ativa no cliente, acessa-se o dashboard pelo IP (ou nome) da mesh do Raspberry.

**3. HTTPS**  
- **Papel do reverse proxy aqui:** O navegador exige HTTPS para certos recursos (ex.: Secure Context do chat). O Nginx, nesse cenário, serve para fornecer o certificado SSL (“cadeado verde”), fazendo o dashboard funcionar sem depender do túnel SSH.
- **Cloudflare Tunnel:** Fornece automaticamente certificado SSL, nao sendo necessário a instalação no servidor.

---

### Opção 1: Túnel SSH (simula localhost)

Esta é a forma mais rápida de testar sem configurar HTTPS: o seu computador (ex.: Galaxy Book) "enxerga" o OpenClaw como se estivesse rodando nele.

No seu Galaxy Book, abra o terminal

Rode este comando (substitua `SEU_USUARIO` e `IP_DO_RASPBERRY` pelos seus dados):

```bash
ssh -L 18789:localhost:18789 SEU_USUARIO@IP_DO_RASPBERRY
```

**Importante:** Mantenha essa janela do terminal aberta. Enquanto o túnel SSH estiver ativo, você consegue acessar o dashboard em `http://localhost:18789`. Se fechar o túnel, o acesso deixa de funcionar até você abrir o túnel novamente.

Agora, no navegador do Galaxy Book, acesse: [http://localhost:18789](http://localhost:18789).

Resultado: O navegador verá "localhost" na URL, ativará o modo seguro automaticamente e o erro sumirá. O tráfego será tunelado via SSH para o Raspberry.

O que exatamente esse comando faz?
O comando `ssh -L 18789:localhost:18789 SEU_USUARIO@IP_DO_RASPBERRY` cria um Túnel SSH com Redirecionamento de Porta Local (Local Port Forwarding).

ssh: Abre o protocolo de comunicação segura.

-L: Diz que você quer fazer um redirecionamento Local.

18789 (o primeiro): É a porta que será aberta no seu Galaxy Book.

localhost:18789 (o meio): Diz que o tráfego que chegar na porta do Galaxy deve ser enviado para o localhost do destino (o Raspberry), na porta 18789.

`SEU_USUARIO@IP_DO_RASPBERRY`: substitua pelo seu usuário SSH e pelo IP do Raspberry (ex.: `usuario@192.168.10.23`).

Em resumo: O seu Galaxy Book reserva a porta 18789 dele. Tudo o que você digita no navegador do Galaxy apontando para localhost:18789 é empacotado, criptografado, enviado pelo cabo de rede até o Raspberry e entregue diretamente ao OpenClaw. Para o navegador, os dados nunca saíram do seu computador, por isso ele ativa o "Secure Context".

---

### Opção 2: HTTPS com reverse proxy (Nginx) com acesso a IP/DNS locais

O **Nginx como reverse proxy** é importante neste cenário por três razões principais: (1) fornece o certificado TLS/SSL que habilita HTTPS no dashboard, eliminando a dependência do túnel SSH para o navegador ativar o "Secure Context" necessário ao chat; (2) permite acessar o OpenClaw diretamente pelo IP local ou hostname do Raspberry (ex.: `https://192.168.10.23:18789` ou `https://raspberry-server.local:18789`) de qualquer dispositivo na rede sem etapas manuais adicionais; (3) serve como camada de terminação TLS, isolando o serviço interno de conexões externas.

> **Não configurado neste projeto.** O Raspberry Pi 3 B+ com 1 GB de RAM mostrou-se insuficiente para rodar OpenClaw de forma estável. Adicionar o Nginx ao mesmo hardware tornaria a situação ainda mais crítica. Esta etapa será documentada em um ambiente com mais recursos (VPS com no mínimo 2 GB de RAM).

---

### Opção 3: HTTPS com Cloudflare Tunnel e/ou reverse proxy (Nginx) com acesso a IP/DNS públicos.

O **Cloudflare Tunnel** é a opção mais segura para acesso público: o Raspberry "disca" para os servidores da Cloudflare sem que nenhuma porta seja aberta no roteador, e o HTTPS com certificado válido é fornecido automaticamente pelo próprio túnel. O Nginx pode atuar em conjunto como reverse proxy interno, recebendo as requisições vindas do túnel e as repassando ao OpenClaw — essa combinação permite expor o dashboard com um domínio próprio (ex.: `https://openclaw.seudominio.com`) de qualquer lugar do mundo, sem IP fixo e sem configuração manual de SSL.

> **Não configurado neste projeto.** Pelo mesmo motivo da Opção 2: o hardware (Raspberry Pi 3 B+ com 1 GB de RAM) não comportava de forma estável nem o OpenClaw sozinho. Esta etapa será documentada em um ambiente com mais recursos (VPS com no mínimo 2 GB de RAM).

---

## Login, aprovação de dispositivo e uso do chat

**Na Opção 1**, acesse `http://localhost:18789/overview`. Na tela de visão geral, faça login: como foi escolhida autenticação por senha no onboarding, use o campo **Password** e cole a senha do gateway. (Ver [tela de login](../../assets/openclaw-dashboard-login.png).)

É necessário aprovar o dispositivo no Raspberry. No terminal do Raspberry:

1. Liste os dispositivos pendentes:
  ```bash
   docker exec -it open-claw openclaw devices list
  ```
2. Aprove usando o ID que aparecer na lista (o ID correto é exibido por `openclaw devices list`; o exemplo abaixo é apenas placeholder):
  ```bash
   docker exec -it open-claw openclaw devices approve xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
  ```

Depois de aprovar, na interface web clique em **Conectar** e veja que no canto superior o icone ira ficar verde

---

## Adicionar e remover modelos

O comando `models set` adiciona modelos ao conjunto disponível. Exemplos:

```bash
docker exec -it open-claw openclaw models set 'openrouter/google/gemini-2.5-flash-lite'
docker exec -it open-claw openclaw models set 'openrouter/deepseek/deepseek-v3.2'
docker exec -it open-claw openclaw models set 'openrouter/openai/gpt-oss-120b:free'
docker exec -it open-claw openclaw models set 'openrouter/meta-llama/llama-3.3-70b-instruct:free'
```

Para remover modelos, edite o arquivo de configuração e apague as entradas desejadas em `agents.defaults.models`:

```bash
sudo nano ~/openclaw/volume/config.toml
```

Por fim, teste o chat, enviando um "Olá" e veja se há resposta.

Se aparecer erro 404 ou falha ao interagir: verifique no painel lateral em Agentes se o modelo que você definiu nas configurações está selecionado para o agente em uso. (Ver [painel de agentes/modelo](../../assets/openclaw-agents-model-panel.png) como exemplo.)

---


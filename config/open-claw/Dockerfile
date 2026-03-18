# Estágio Base: Node 24 (Recomendado)
FROM node:24-bookworm-slim AS base

# Estágio de Instalação
FROM base AS installer
RUN apt-get update && apt-get install -y curl bash procps git
WORKDIR /install

# Instalamos o pacote globalmente em uma pasta específica para podermos copiar depois
RUN npm install -g openclaw@2026.3.11 --prefix /root/.openclaw

# ---------

# Estágio de Execução (Runner)
FROM base AS runner
WORKDIR /app

# Criando grupo e usuário conforme o padrão de segurança (1001)
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 app

# Copiamos os arquivos instalados no estágio anterior
COPY --from=installer --chown=app:nodejs /root/.openclaw/bin /app/bin
COPY --from=installer --chown=app:nodejs /root/.openclaw/lib/node_modules /app/lib/node_modules

# Variáveis de ambiente para persistência
ENV OPENCLAW_HOME=/app/data
ENV OPENCLAW_STATE_DIR=/app/data/state
ENV OPENCLAW_CONFIG_PATH=/app/data/config.toml
ENV PATH="/app/bin:${PATH}"
ENV NODE_PATH=/app/lib/node_modules

# Criar pastas de dados e garantir permissões internas
RUN mkdir -p /app/data /app/bin && chown -R app:nodejs /app

# Mudar para o usuário seguro
USER app

# Porta do Dashboard/Control UI
EXPOSE 18789

# Comando para rodar o gateway em modo foreground
ENTRYPOINT ["openclaw", "gateway", "run", "--allow-unconfigured"]

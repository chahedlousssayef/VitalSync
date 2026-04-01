# =============================================================================
# Multi-stage build — VitalSync Backend Node.js
#
# Pourquoi le multi-stage build ?
# - Sécurité : les outils de dev (jest, eslint, npm) ne sont PAS présents en
#   production. Moins d'outils = surface d'attaque réduite.
# - Taille : l'image finale ne contient que le runtime Node.js + le code app.
#   On passe typiquement de ~900 Mo (image complète) à ~150 Mo (alpine).
# =============================================================================
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./

RUN npm ci --include=dev
COPY . .

RUN npm test

FROM node:20-alpine AS production

WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=builder /app/server.js ./

USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
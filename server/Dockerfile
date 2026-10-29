FROM node:24-trixie-slim AS build

WORKDIR /usr/app

COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund

COPY . .
RUN npm run build
RUN npm prune --omit=dev

# Stage 2: Production runtime
FROM gcr.io/distroless/nodejs24-debian13:nonroot AS runtime
WORKDIR /usr/app
ENV NODE_ENV=production

COPY --from=build --chown=nonroot:nonroot /usr/app/package.json ./package.json
COPY --from=build --chown=nonroot:nonroot /usr/app/node_modules ./node_modules
COPY --from=build --chown=nonroot:nonroot /usr/app/dist ./dist

CMD ["dist/server.js"]

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit-dev

FROM node:20-alpine AS runtime
RUN addgroup -S app && adduser -S appuser -G app
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER appuser
CMD ["node", "src/worker.js"]
# cn-backend — Express + TypeScript + Prisma (PostgreSQL)
FROM node:22-alpine AS deps
RUN apk add --no-cache libc6-compat openssl
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

FROM node:22-alpine AS builder
RUN apk add --no-cache libc6-compat openssl
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY package.json package-lock.json ./
COPY tsconfig.json ./
COPY prisma ./prisma
COPY src ./src
ENV DATABASE_URL="postgresql://build:build@127.0.0.1:5432/build?schema=public"
# नोट: हमने यहाँ से --omit=dev हटा दिया है ताकि रनटाइम पैकेजेस मिस न हों
RUN npx prisma generate && npm run build

FROM node:22-alpine AS runner
RUN apk add --no-cache libc6-compat openssl wget
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs \
  && adduser --system --uid 1001 expressjs

# बिल्डर स्टेज से सारा ज़रूरी डेटा कॉपी कर रहे हैं
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma

RUN mkdir -p uploads && chown -R expressjs:nodejs /app/uploads

USER expressjs

ENV PORT=4000
EXPOSE 4000

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- "http://127.0.0.1:${PORT:-4000}/health" || exit 1

CMD ["node", "dist/index.js"]
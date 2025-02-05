FROM node:22-alpine AS base
WORKDIR /app
ENV NEXT_TELEMETRY_DISABLED=1

#=============================================================================#

FROM base AS deps
RUN apk add --no-cache libc6-compat
COPY package.json pnpm-lock.yaml ./
RUN npm i -g corepack@latest && corepack enable pnpm && pnpm i --frozen-lockfile

#=============================================================================#

FROM deps AS builder
COPY . .
RUN npm i -g corepack@latest && corepack enable pnpm && pnpm run build

#=============================================================================#

FROM base AS runner
ENV NODE_ENV=production
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

ENV HOSTNAME=0.0.0.0
ENV PORT=3000
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://0.0.0.0:3000/api/health || exit 1

CMD ["node", "server.js"]

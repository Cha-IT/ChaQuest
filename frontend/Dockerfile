# Frontend Dockerfile (Next.js)
# Multi-stage to keep runtime image small and allow caching on package changes

FROM node:20-alpine AS deps
WORKDIR /app
# Install OS deps needed for sharp/next if any
RUN apk add --no-cache libc6-compat
COPY package.json package-lock.json ./
RUN npm ci --legacy-peer-deps

FROM node:20-alpine AS builder
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# Generate Prisma client (required by app imports)
RUN npx prisma generate
# Build Next app
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001

# Copy only necessary files
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
# Prisma client runtime (if used on server)
COPY --from=builder /app/prisma ./prisma

EXPOSE 3100
USER nextjs
CMD ["npm", "run", "start", "--", "-p", "3100"]

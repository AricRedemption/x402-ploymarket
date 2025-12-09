# Build stage
FROM node:20-alpine AS builder

# Install pnpm
RUN corepack enable && corepack prepare pnpm@8.15.0 --activate

WORKDIR /app

# Copy workspace files
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./
COPY packages/shared-types ./packages/shared-types
COPY prediction-market-back-end ./prediction-market-back-end

# Install dependencies
RUN pnpm install --frozen-lockfile

# Build shared-types first, then backend
RUN pnpm --filter @x402/shared-types build
RUN pnpm --filter prediction-market-back-end build

# Production stage
FROM node:20-alpine AS runner

RUN corepack enable && corepack prepare pnpm@8.15.0 --activate

WORKDIR /app

ENV NODE_ENV=production

# Copy workspace files
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./
COPY packages/shared-types/package.json ./packages/shared-types/
COPY prediction-market-back-end/package.json ./prediction-market-back-end/

# Install production dependencies only
RUN pnpm install --frozen-lockfile --prod

# Copy built files
COPY --from=builder /app/packages/shared-types/dist ./packages/shared-types/dist
COPY --from=builder /app/prediction-market-back-end/dist ./prediction-market-back-end/dist

# Copy IDL file needed at runtime
COPY prediction-market-back-end/src/blockchain/solana/idl ./prediction-market-back-end/dist/blockchain/solana/idl

WORKDIR /app/prediction-market-back-end

EXPOSE 3001

CMD ["node", "dist/index.js"]

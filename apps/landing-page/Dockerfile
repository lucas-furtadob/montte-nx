# apps/landing-page/Dockerfile
FROM oven/bun:1.2.23 AS builder




WORKDIR /app

# Copy all package manifests and the lockfile first.
# This takes advantage of Docker's layer caching, speeding up subsequent builds.
COPY bun.lock ./
COPY package.json ./
# apps
COPY apps/dashboard/package.json ./apps/dashboard/
COPY apps/landing-page/package.json ./apps/landing-page/
COPY apps/server/package.json ./apps/server/
COPY apps/worker/package.json ./apps/worker/
# libraries
COPY libraries/condition-evaluator/package.json ./libraries/condition-evaluator/
COPY libraries/csv/package.json ./libraries/csv/
COPY libraries/money/package.json ./libraries/money/
COPY libraries/ofx/package.json ./libraries/ofx/
COPY libraries/rules-engine/package.json ./libraries/rules-engine/
# packages
COPY packages/analytics/package.json ./packages/analytics/
COPY packages/api/package.json ./packages/api/
COPY packages/arcjet/package.json ./packages/arcjet/
COPY packages/authentication/package.json ./packages/authentication/
COPY packages/brasil-api/package.json ./packages/brasil-api/
COPY packages/cache/package.json ./packages/cache/
COPY packages/csv/package.json ./packages/csv/
COPY packages/database/package.json ./packages/database/
COPY packages/encryption/package.json ./packages/encryption/
COPY packages/environment/package.json ./packages/environment/
COPY packages/files/package.json ./packages/files/
COPY packages/logging/package.json ./packages/logging/
COPY packages/money/package.json ./packages/money/
COPY packages/notifications/package.json ./packages/notifications/
COPY packages/ofx/package.json ./packages/ofx/
COPY packages/pdf/package.json ./packages/pdf/
COPY packages/posthog/package.json ./packages/posthog/
COPY packages/queue/package.json ./packages/queue/
COPY packages/reconciliation/package.json ./packages/reconciliation/
COPY packages/stripe/package.json ./packages/stripe/
COPY packages/transactional/package.json ./packages/transactional/
COPY packages/ui/package.json ./packages/ui/
COPY packages/utils/package.json ./packages/utils/
COPY packages/workflows/package.json ./packages/workflows/
# tooling
COPY tooling/typescript/package.json ./tooling/typescript/

# Install all dependencies (including devDependencies needed for the build step)
RUN bun install

# Copy the rest of the source code for all workspaces.
# A .dockerignore file in your monorepo root is crucial for this step.
COPY . .

# Build the landing page application.
# The build output will be in /app/apps/landing-page/dist
ENV NODE_ENV=production
ARG VITE_POSTHOG_HOST
ARG VITE_POSTHOG_KEY
ARG VITE_POSTHOG_UI_HOST

RUN bun run --filter=landing-page build

# ---

# Stage 2: Serve with nginx
FROM nginx:alpine AS runner

# Copy the built static files from the builder stage
COPY --from=builder /app/apps/landing-page/dist /usr/share/nginx/html

# Copy custom nginx configuration and entrypoint script
COPY apps/landing-page/nginx.conf /etc/nginx/nginx.conf
COPY apps/landing-page/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Set and expose the port. Default to 4321 if not provided.

ARG PORT

ENV PORT=${PORT}
EXPOSE ${PORT}

# Start nginx with entrypoint script
CMD ["/entrypoint.sh"]

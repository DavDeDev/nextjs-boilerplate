# use the official Bun image
# see all versions at https://hub.docker.com/r/oven/bun/tags
FROM oven/bun:latest as base
WORKDIR /usr/src/app
ENV NEXT_TELEMETRY_DISABLED 1

# install dependencies into temp directory
# this will cache them and speed up future builds
FROM base AS install
WORKDIR /usr/src/app
RUN mkdir -p /temp/deps
COPY package.json bun.lockb /temp/deps/
RUN cd /temp/deps && bun install --frozen-lockfile

# copy node_modules from temp directory
# then copy all (non-ignored) project files into the image
FROM base AS prerelease
COPY --from=install /temp/deps/node_modules node_modules
COPY . .

# [optional] tests & build
ENV NODE_ENV=production
RUN bun test
RUN bun run build

# copy production dependencies and source code into final image
FROM base AS release

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=prerelease --chown=nextjs:nodejs /usr/src/app/.next/standalone ./
COPY --from=prerelease --chown=nextjs:nodejs /usr/src/app/public ./public
COPY --from=prerelease --chown=nextjs:nodejs /usr/src/app/.next/static ./.next/static/

EXPOSE 3000/tcp
ENTRYPOINT [ "bun","server.js" ]
# # run the app
# # USER bun
# ENTRYPOINT [ "bun", "run", "index.ts" ]



# FROM node:18-alpine AS base

# # Step 1. Rebuild the source code only when needed
# FROM base AS builder

# WORKDIR /app

# # Install dependencies based on the preferred package manager
# COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
# # Omit --production flag for TypeScript devDependencies
# RUN \
#   if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
#   elif [ -f package-lock.json ]; then npm ci; \
#   elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm i; \
#   # Allow install without lockfile, so example works even without Node.js installed locally
#   else echo "Warning: Lockfile not found. It is recommended to commit lockfiles to version control." && yarn install; \
#   fi

# COPY src ./src
# COPY public ./public
# COPY next.config.js .
# COPY tsconfig.json .

# # Environment variables must be present at build time
# # https://github.com/vercel/next.js/discussions/14030
# ARG ENV_VARIABLE
# ENV ENV_VARIABLE=${ENV_VARIABLE}
# ARG NEXT_PUBLIC_ENV_VARIABLE
# ENV NEXT_PUBLIC_ENV_VARIABLE=${NEXT_PUBLIC_ENV_VARIABLE}

# # Next.js collects completely anonymous telemetry data about general usage. Learn more here: https://nextjs.org/telemetry
# # Uncomment the following line to disable telemetry at build time
# # ENV NEXT_TELEMETRY_DISABLED 1

# # Build Next.js based on the preferred package manager
# RUN \
#   if [ -f yarn.lock ]; then yarn build; \
#   elif [ -f package-lock.json ]; then npm run build; \
#   elif [ -f pnpm-lock.yaml ]; then pnpm build; \
#   else npm run build; \
#   fi

# # Note: It is not necessary to add an intermediate step that does a full copy of `node_modules` here

# # Step 2. Production image, copy all the files and run next
# FROM base AS runner

# WORKDIR /app

# # Don't run production as root
# RUN addgroup --system --gid 1001 nodejs
# RUN adduser --system --uid 1001 nextjs
# USER nextjs

# COPY --from=builder /app/public ./public

# # Automatically leverage output traces to reduce image size
# # https://nextjs.org/docs/advanced-features/output-file-tracing
# COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
# COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

# # Environment variables must be redefined at run time
# ARG ENV_VARIABLE
# ENV ENV_VARIABLE=${ENV_VARIABLE}
# ARG NEXT_PUBLIC_ENV_VARIABLE
# ENV NEXT_PUBLIC_ENV_VARIABLE=${NEXT_PUBLIC_ENV_VARIABLE}

# # Uncomment the following line to disable telemetry at run time
# # ENV NEXT_TELEMETRY_DISABLED 1

# # Note: Don't expose ports here, Compose will handle that for us

# CMD ["node", "server.js"]
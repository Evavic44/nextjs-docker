## Docker Setup

Steps to use docker with nextjs.

> **Note**
> This example has been optimized to work with npm only, if you're using yarn, follow the original guide: [Nexjs Docker Setup](https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile)

- Create a `Dockerfile` in project root.
- Configure options using the below code:

```sh
FROM node:18-alpine AS base
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package*.json ./
RUN npm install

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public

RUN mkdir .next
RUN chown nextjs:nodejs .next

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"
CMD ["node", "server.js"]
```

### Enable Nextjs Trace copying

```js
// nextjs.config.js
module.exports = {
  output: "standalone",
};
```

[Source:](https://nextjs.org/docs/app/api-reference/next-config-js/output#automatically-copying-traced-files)

### Run Docker Image

Run dockerfile using the command:

```sh
docker build -t app-name .
# example: docker build -t nextjs-docker .
```

Start image in local server using

```sh
docker run -e PORT=3000 project-name

# or
docker compose up --build
# example: docker run -e PORT=3000 nextjs-docker
```

### Resources

- [Containerize an application](https://docs.docker.com/get-started/02_our_app/)

FROM node:16-alpine AS builder
ENV APP_NAME=web1
RUN apk update
WORKDIR /app
RUN yarn global add turbo
COPY . .
RUN turbo prune --scope=$APP_NAME --docker

# Add lockfile and package.json's of isolated subworkspace
FROM node:16-alpine AS installer
ENV APP_NAME=web1
RUN apk update
WORKDIR /app
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/yarn.lock ./yarn.lock
RUN yarn install
COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json
RUN yarn turbo run build --filter=$APP_NAME

FROM node:16-alpine AS runner
WORKDIR /app
ENV APP_NAME=web1
COPY --from=installer /app/apps/$APP_NAME/next.config.js .
COPY --from=installer /app/apps/$APP_NAME/package.json .
COPY --from=installer /app/apps/$APP_NAME/.next/standalone ./
COPY --from=installer /app/apps/$APP_NAME/.next/static ./apps/$APP_NAME/.next/static
CMD node apps/$APP_NAME/server.js

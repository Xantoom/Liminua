#syntax=docker/dockerfile:1

# use the official Bun image
# see all versions at https://hub.docker.com/r/oven/bun/tags
FROM oven/bun:1 AS bun_base

WORKDIR /usr/src/app

# install dependencies into temp directory
# this will cache them and speed up future builds
RUN mkdir -p /temp/dev
COPY package.json bun.lockb* /temp/dev/
RUN cd /temp/dev && bun install --frozen-lockfile

# install with --production (exclude devDependencies)
RUN mkdir -p /temp/prod
COPY package.json bun.lockb* /temp/prod/
RUN cd /temp/prod && bun install --frozen-lockfile --production

# Dev Bun image
FROM bun_base AS bun_dev

ENV NODE_ENV=development

# copy dev dependencies
COPY --from=bun_base /temp/dev/node_modules node_modules

# copy all source files
COPY . .

# run with hot reload
USER bun
EXPOSE 3000/tcp
CMD ["bun", "--hot", "run", "src/index.ts"]

# Prod Bun image
FROM bun_base AS bun_prod

ENV NODE_ENV=production

# copy production dependencies
COPY --from=bun_base /temp/prod/node_modules node_modules

# copy source code
COPY . .

# [optional] run tests and build
RUN bun test
RUN bun run build

# run the app
USER bun
EXPOSE 3000/tcp
ENTRYPOINT ["bun", "run", "src/index.ts"]

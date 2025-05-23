FROM node:current-alpine AS base
RUN apk add -U build-base python3 py3-setuptools
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable
RUN npx pnpm i -g pnpm@latest
WORKDIR /usr/src/app
COPY . /usr/src/app

FROM base AS prod-deps
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --prod --frozen-lockfile

FROM base AS build
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
RUN pnpm run build

FROM node:current-alpine
LABEL maintainer="butlerx@notthe.cloud"
WORKDIR /usr/src/app
ENV NODE_ENV=production
COPY --from=prod-deps /usr/src/app/node_modules /usr/src/app/node_modules
COPY --from=build /usr/src/app/build /usr/src/app/build

# FIXME
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
COPY --from=base /pnpm /pnpm

COPY package.json /usr/src/app
RUN apk add -U coreutils openssh-client sshpass && \
    mkdir ~/.ssh

EXPOSE 3000
CMD [ "pnpm", "start" ]

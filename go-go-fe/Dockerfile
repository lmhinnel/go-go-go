FROM --platform=$BUILDPLATFORM node:18.3.0-alpine AS development

RUN mkdir /project
WORKDIR /project

COPY . .

COPY ./.env.example ./.env

RUN yarn global add @vue/cli
RUN yarn install
ENV HOST=0.0.0.0

CMD ["yarn", "run", "serve"]

FROM development as dev-envs
RUN <<EOF
apk update
apk add git
EOF

RUN <<EOF
addgroup -S docker
adduser -S --shell /bin/bash --ingroup docker vscode
EOF

# install Docker tools (cli, buildx, compose)

COPY --from=gloursdocker/docker / /
CMD ["yarn", "run", "serve"]

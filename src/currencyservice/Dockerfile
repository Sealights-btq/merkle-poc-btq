FROM --platform=$BUILDPLATFORM node:22-alpine AS builder

ARG SEALIGHTS_TOKEN
ARG BUILD_NAME
ENV SEALIGHTS_NODE_AGENT=slnodejs
ARG BRANCH

# Some packages (e.g. @google-cloud/profiler) require additional
# deps for post-install scripts
RUN apk add --update --no-cache \
    python3 \
    make \
    g++

# Install pnpm globally
RUN npm install -g pnpm@9

WORKDIR /usr/src/app

COPY package*.json ./
RUN pnpm install --prod

#install agent (Not a url)
RUN pnpm add ${SEALIGHTS_NODE_AGENT}

FROM node:22-alpine

# Install pnpm globally
RUN npm install -g pnpm@9

WORKDIR /usr/src/app

COPY --from=builder /usr/src/app/node_modules ./node_modules

COPY . .
#run config to get build session id
RUN  pnpm exec slnodejs config --token ${SEALIGHTS_TOKEN} --appname "currencyservice" --branch ${BRANCH} --build ${BUILD_NAME}
#sl-scan, scanning the code
RUN pnpm exec slnodejs scan --token ${SEALIGHTS_TOKEN} --buildsessionidfile buildSessionId --workspacepath "." --scm none --es6Modules
ENV NODE_DEBUG=sl

EXPOSE 7000
#Running the code with sealights
CMD  pnpm exec slnodejs run --token "${SL_TOKEN}" --buildsessionidfile buildSessionId --labid "${SL_LAB_ID}" --workspacepath . server.js

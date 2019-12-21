#############################################################################
################################ build stage ################################
#############################################################################

FROM node:current-alpine AS builder

WORKDIR /usr/src/app

RUN \
    apk add --no-cache                               \
        build-base                                   \
        git                                          \
        python;                                      \
    git clone https://github.com/jfurrow/flood.git .

COPY config.js /usr/src/app/config.js

RUN \
    npm install;  \
    npm run build

################################################################################
################################ run-time stage ################################
################################################################################

FROM node:current-alpine

WORKDIR /usr/src/app

RUN \
    apk add --no-cache \
    curl               \
    mediainfo

COPY --from=builder /usr/src/app /usr/src/app

CMD [ "npm", "run", "start" ]

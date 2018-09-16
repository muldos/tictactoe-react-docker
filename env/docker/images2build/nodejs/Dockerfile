FROM node:latest
MAINTAINER David ROBIN <david.robin@gpsante.fr>

ENV NPM_CONFIG_PREFIX=/home/node/.npm-global
# optionally if you want to run npm global bin without specifying path
ENV PATH=$PATH:/home/node/.npm-global/bin

ENV NPM_CONFIG_LOGLEVEL info
#Modify UID of node into UID of local user
ARG NODE_UID=1000
RUN usermod -u ${NODE_UID} node && groupmod -g ${NODE_UID} node
WORKDIR /home/node
USER node
CMD mkdir -p /home/node/www
#tips pour avoir le container toujours up
USER root
CMD tail -f /dev/null

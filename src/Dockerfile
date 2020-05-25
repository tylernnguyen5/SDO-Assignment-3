FROM node:12.2.0-alpine 
WORKDIR /usr/app

# RUN apk add --no-cache bash

COPY package*.json ./

RUN NODE_ENV=production npm install

COPY . . 

CMD [ "npm", "run", "start:prod" ]
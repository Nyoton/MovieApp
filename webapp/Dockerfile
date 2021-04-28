FROM node:alpine 

WORKDIR /app

COPY . .

RUN npm upgrade -g npm && npm upgrade -g yarn && yarn install 

EXPOSE 80

CMD yarn start

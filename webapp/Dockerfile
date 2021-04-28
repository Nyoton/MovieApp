FROM node:alpine 

WORKDIR /app

COPY . .

RUN npm upgrade -g npm && npm upgrade -g yarn && yarn install 

EXPOSE 9999

CMD yarn start

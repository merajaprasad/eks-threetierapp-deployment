# stage 1
FROM node:14 as build
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .

# stage 2

FROM node:14-slim
WORKDIR /usr/src/app
COPY --from=build  /usr/src/app .
CMD ["node", "index.js"]

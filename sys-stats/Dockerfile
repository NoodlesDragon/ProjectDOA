FROM node:16-alpine

WORKDIR /app

ENV PATH ="./node_modules/.bin:$PATH"

COPY . .

RUN npm install

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]


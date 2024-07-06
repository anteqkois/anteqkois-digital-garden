FROM node:lts-alpine

WORKDIR /app

RUN npm install -g pnpm@8.15.2
COPY package.json .
COPY pnpm-lock.yaml .

RUN pnpm install

COPY . .

RUN npm run build

CMD ["npm", "run", "start"]

# TODO add stages to use builder and reduce final size
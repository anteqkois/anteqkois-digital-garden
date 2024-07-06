FROM node:lts-alpine

WORKDIR /app

RUN npm install -g pnpm@8.15.2
RUN pnpm install
RUN npm run build

CMD ["npm", "run", "start"]
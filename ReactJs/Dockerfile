FROM node:20 AS build

ARG VITE_API_BASE_URL

WORKDIR /app

COPY package*.json ./

RUN yarn

COPY . .

ENV VITE_API_BASE_URL=${VITE_API_BASE_URL}

RUN yarn build

FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
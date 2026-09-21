FROM node:24-slim AS build

WORKDIR /usr/src/app
COPY package.json package-lock.json ./

RUN npm ci --no-audit --no-fund
COPY . .

RUN npm run build --prod

FROM public.ecr.aws/nginx/nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=build /usr/src/app/dist/client/browser/ /usr/share/nginx/html
RUN cp /usr/share/nginx/html/index.csr.html /usr/share/nginx/html/index.html
CMD ["nginx", "-g", "daemon off;"]
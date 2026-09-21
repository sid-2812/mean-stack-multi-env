FROM node:24-slim
WORKDIR /usr/app

# Install dependencies and build the project.
COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund
COPY . .
RUN npm run build

# Run the web service on container startup.
CMD ["node", "dist/server.js"]

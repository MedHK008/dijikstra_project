# Build stage
FROM node:lts as build-stage

WORKDIR /app
COPY dijikstra_frontend/package*.json ./  
RUN npm install
COPY dijikstra_frontend/ .  
RUN npm run build

# Production stage
FROM nginx:stable-alpine
COPY --from=build-stage /app/dist /usr/share/nginx/html
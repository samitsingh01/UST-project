# Stage 1: Build the React app using Node.js
FROM node:18-alpine AS builder

# Set working directory inside the container
WORKDIR /app

# Copy only package.json first (for caching dependencies)
COPY package.json ./

# Install project dependencies
RUN npm install

# Copy the rest of the React source code
COPY public/ ./public/
COPY src/ ./src/

# Set API URL at build time (frontend cannot read runtime env vars directly)
ARG REACT_APP_API_URL=http://localhost:8000
ENV REACT_APP_API_URL=$REACT_APP_API_URL

# Build the optimized production version of React app
RUN npm run build


# Stage 2: Serve the app with Nginx
FROM nginx:1.25-alpine

# Remove default Nginx configs
RUN rm -rf /etc/nginx/conf.d/*

# Copy built React files from builder stage to Nginx web root
COPY --from=builder /app/build /usr/share/nginx/html

# Write custom Nginx config
# - Serves React app
# - Proxies /api requests to API Gateway
# - Adds a /health endpoint
RUN echo 'server { \
    listen 3000; \
    server_name localhost; \
    location / { \
        root /usr/share/nginx/html; \
        index index.html index.htm; \
        try_files $uri $uri/ /index.html; \
    } \
    location /api/ { \
        proxy_pass http://api-gateway:8000/; \
        proxy_set_header Host $host; \
        proxy_set_header X-Real-IP $remote_addr; \
    } \
    location /health { \
        access_log off; \
        return 200 "healthy\n"; \
    } \
}' > /etc/nginx/conf.d/default.conf

# Fix permissions so nginx user can read/write properly
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid

# Run as non-root user (safer)
USER nginx

# Expose port 3000 for serving frontend
EXPOSE 3000

# Start Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]

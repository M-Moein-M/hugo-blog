# Build stage - build the Hugo site
FROM hugomods/hugo:latest AS builder

WORKDIR /src
COPY . .

# Build the site
RUN hugo --minify

# Production stage - serve with nginx
FROM nginx:alpine

# Copy the built site from the builder stage
COPY --from=builder /src/public/ /usr/share/nginx/html/

EXPOSE 80


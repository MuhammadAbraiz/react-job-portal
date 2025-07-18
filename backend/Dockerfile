# Build stage
FROM node:18-alpine AS build

WORKDIR /app

# Install build dependencies
RUN apk add --no-cache python3 make g++

# Copy package files first to leverage Docker cache
COPY backend/package*.json ./

# Install all dependencies (including devDependencies)
RUN npm install

# Copy source code
COPY backend/ .

# Production stage
FROM node:18-alpine

# Create app directory and set permissions
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    mkdir -p /app && \
    chown -R nodejs:nodejs /app

WORKDIR /app

# Install runtime dependencies
RUN apk add --no-cache curl

# Copy package files and install production dependencies
COPY --from=build --chown=nodejs:nodejs /app/package*.json ./

# Install production dependencies and clean cache in one layer
RUN npm install --only=production && \
    npm cache clean --force

# Copy application code with correct permissions
COPY --from=build --chown=nodejs:nodejs /app/ .

# Switch to non-root user
USER nodejs

# Expose the app port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# Command to run the application
CMD ["node", "server.js"]
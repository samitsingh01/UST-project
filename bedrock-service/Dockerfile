# Stage 1: Build the Go app
FROM golang:1.21-alpine AS builder

# Install tools needed for building Go code
RUN apk add --no-cache git ca-certificates

# Set working directory
WORKDIR /app

# Copy dependency file first for caching
COPY go.mod ./

# Copy main application code
COPY main.go .

# Download Go dependencies
RUN go mod download || true
RUN go mod tidy

# Build a static binary (no external dependencies)
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o bedrock-service .


# Stage 2: Minimal runtime container
FROM alpine:3.19

# Install CA certs for HTTPS requests
RUN apk --no-cache add ca-certificates

# Create non-root user for security
RUN addgroup -g 1000 -S appuser && \
    adduser -u 1000 -S appuser -G appuser

# Set working directory
WORKDIR /app

# Copy compiled Go binary from builder stage
COPY --from=builder /app/bedrock-service .

# Give ownership of files to non-root user
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Expose port 9000 for this service
EXPOSE 9000

# Health check endpoint (K8s/Docker will use this)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:9000/health || exit 1

# Run the Go service
CMD ["./bedrock-service"]

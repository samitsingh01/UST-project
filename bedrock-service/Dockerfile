# Build Go app
FROM golang:1.21-alpine AS builder
WORKDIR /app

COPY go.mod ./
COPY main.go ./
RUN go mod tidy
RUN go build -o bedrock-service .

# Minimal runtime
FROM alpine:3.19
WORKDIR /app

COPY --from=builder /app/bedrock-service .

EXPOSE 9000
CMD ["./bedrock-service"]

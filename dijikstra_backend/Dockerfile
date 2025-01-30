# Stage 1: Build
FROM gcc:12 AS builder

WORKDIR /app
COPY . .
RUN apt-get update && apt-get install -y cmake ninja-build
RUN mkdir build && cd build && cmake -G Ninja .. && cmake --build .

# Stage 2: Runtime
FROM debian:bullseye-slim
WORKDIR /app
COPY --from=builder /app/build/Dijikstra_backend .
EXPOSE 8080
CMD ["./Dijikstra_backend"]
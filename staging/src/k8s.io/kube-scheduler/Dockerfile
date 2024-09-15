# First stage: Build the kube-scheduler binary
FROM golang:1.20-alpine AS builder

# Install necessary build dependencies
RUN apk add --no-cache make bash

# Set environment variables for Go
ENV GO111MODULE=on
ENV GOPATH=/go
ENV PATH=$GOPATH/bin:/usr/local/go/bin:$PATH

# Set the working directory inside the container
WORKDIR /go/src/k8s.io/kubernetes

# Copy the already checked-out Kubernetes code from the action
COPY ../../../../ .

# Build kube-scheduler binary
RUN make WHAT=cmd/kube-scheduler

# Second stage: Create a minimal Alpine-based image
ARG BASE_REGISTRY=201959883603.dkr.ecr.us-east-2.amazonaws.com
ARG BASE_IMAGE_PATH=mdaca/base-images/ironbank-alpine
ARG BASE_TAG=3.20.2
ARG BASE_IMAGE=${BASE_REGISTRY}/${BASE_IMAGE_PATH}:${BASE_TAG}
ARG $BASE_IMAGE
FROM $BASE_IMAGE as final

# Install runtime dependencies
RUN apk add --no-cache ca-certificates

# Create a non-root user and group
RUN addgroup -S schedulergroup && adduser -S scheduleruser -G schedulergroup

# Set the working directory
WORKDIR /usr/local/bin

# Copy the kube-scheduler binary from the builder stage and adjust permissions
COPY --from=builder /go/src/k8s.io/kubernetes/_output/bin/kube-scheduler /usr/local/bin/kube-scheduler

# Ensure the non-root user has ownership and execution permissions on the binary
RUN chown scheduleruser:schedulergroup /usr/local/bin/kube-scheduler && \
    chmod +x /usr/local/bin/kube-scheduler

# Expose kube-scheduler ports
EXPOSE 10259 10257

# Switch to the non-root user
USER scheduleruser

# Command to run kube-scheduler
ENTRYPOINT ["kube-scheduler"]

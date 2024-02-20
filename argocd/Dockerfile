# Use Alpine Linux as the base image for its small footprint
FROM alpine:3.19.1

# Define the version of Argo CD CLI you want to install
ARG ARGOCD_VERSION=v2.10.1

# Install necessary packages
RUN apk add --no-cache curl tar

# Download and install the Argo CD CLI
RUN curl -sSL "https://github.com/argoproj/argo-cd/releases/download/${ARGOCD_VERSION}/argocd-linux-amd64" -o /usr/local/bin/argocd && \
    chmod +x /usr/local/bin/argocd

# Set the entrypoint to the Argo CD CLI
ENTRYPOINT [ "argocd" ]

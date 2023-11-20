FROM alpine:3.14.2

RUN apk add --no-cache \
    bash \
    curl \
    jq

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" \
    && unzip awscliv2.zip \
    && ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update \
    && rm -rf awscliv2.zip ./aws

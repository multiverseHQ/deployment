FROM alpine:3.5

ENV PROXY_IP 0.0.0.0

ENV VALIDATORS 0.0.0.0
ENV SEEDS 0.0.0.0



# This is the release of tendermint to pull in.
ENV TM_VERSION 0.10.0
ENV TM_SHA256SUM a29852b8d51c00db93c87c3d148fa419a047abd38f32b2507a905805131acc19

# Tendermint will be looking for genesis file in /tendermint (unless you change
# `genesis_file` in config.toml). You can put your config.toml and private
# validator file into /tendermint.
#
# The /tendermint/data dir is used by tendermint to store state.
ENV DATA_ROOT /tendermint
ENV TMHOME $DATA_ROOT

# Set user right away for determinism
RUN addgroup tmuser && \
    adduser -S -G tmuser tmuser

# Create directory for persistence and give our user ownership
RUN mkdir -p $DATA_ROOT && \
    chown -R tmuser:tmuser $DATA_ROOT

# jq and curl used for extracting `pub_key` from private validator while
# deploying tendermint with Kubernetes. It is nice to have bash so the users
# could execute bash commands.
RUN apk add --no-cache bash curl jq

RUN apk add --no-cache openssl && \
    wget https://s3-us-west-2.amazonaws.com/tendermint/binaries/tendermint/v${TM_VERSION}/tendermint_${TM_VERSION}_linux_amd64.zip && \
    echo "${TM_SHA256SUM}  tendermint_${TM_VERSION}_linux_amd64.zip" | sha256sum -c && \
    unzip -d /bin tendermint_${TM_VERSION}_linux_amd64.zip && \
    apk del openssl && \
    rm -f tendermint_${TM_VERSION}_linux_amd64.zip

COPY ./genesis-template.json /

RUN chmod 777 /genesis-template.json

COPY ./tendermint-entrypoint.sh /

RUN chmod 777 /tendermint-entrypoint.sh

# Expose the data directory as a volume since there's mutable state in there
VOLUME $DATA_ROOT

# p2p port
EXPOSE 46656
# rpc port
EXPOSE 46657

CMD ["/tendermint-entrypoint.sh"]

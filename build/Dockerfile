# Build the mattermost cloud
ARG DOCKER_BUILD_IMAGE=golang:1.14.1
ARG DOCKER_BASE_IMAGE=alpine:3.11.3

FROM ${DOCKER_BUILD_IMAGE} AS build
WORKDIR /mattermost-cloud/
COPY . /mattermost-cloud/
RUN apt-get update -yq && apt-get install -yq unzip
RUN make get-terraform get-kops get-helm get-kubectl
RUN make build

# Final Image
FROM ${DOCKER_BASE_IMAGE}
LABEL name="Mattermost Cloud" \
  maintainer="cloud-team@mattermost.com" \
  vendor="Mattermost" \
  distribution-scope="public" \
  architecture="x86_64" \
  url="https://mattermost.com" \
  io.k8s.description="Mattermost Cloud creates, configures and helps manage K8s Clusters and Mattermost installations on Kubernetes" \
  io.k8s.display-name="Mattermost Cloud"

ENV CLOUD=/mattermost-cloud/cloud \
    USER_UID=10001 \
    USER_NAME=cloud

RUN  apk update && apk add libc6-compat && apk add ca-certificates
COPY --from=build /mattermost-cloud/build/terraform /usr/local/bin/
COPY --from=build /mattermost-cloud/build/kops /usr/local/bin/
COPY --from=build /mattermost-cloud/build/helm /usr/local/bin/
COPY --from=build /mattermost-cloud/build/kubectl /usr/local/bin/
COPY --from=build /mattermost-cloud/operator-manifests /mattermost-cloud/operator-manifests
COPY --from=build /mattermost-cloud/helm-charts /mattermost-cloud/helm-charts
COPY --from=build /mattermost-cloud/terraform /mattermost-cloud/terraform
COPY --from=build /mattermost-cloud/build/_output/bin/cloud /mattermost-cloud/cloud
COPY --from=build /mattermost-cloud/build/bin /usr/local/bin
RUN  /usr/local/bin/user_setup
WORKDIR /mattermost-cloud/

ENTRYPOINT ["/usr/local/bin/entrypoint"]

USER ${USER_UID}

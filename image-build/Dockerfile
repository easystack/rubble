# 该镜像就是前面的cilium 适配rubble的build镜像
# ARG TERWAY_POLICY_IMAGE=registry.cn-hongkong.aliyuncs.com/acs/terway:policy-20230413-b0c060f3@sha256:757b30a70fe3aed011ab247107eb177a061cfffa00ce453bcf800c4b69a987d9
ARG RUBBLE_POLICY_IMAGE=rubble:policy-20230606

ARG UBUNTU_IMAGE=docker.io/library/ubuntu:20.04@sha256:b33325a00c7c27b23ae48cf17d2c654e2c30812b35e7846c006389318f6a71c2
ARG CILIUM_LLVM_IMAGE=quay.io/cilium/cilium-llvm:a8c542efc076b62ba683e7699c0013adb6955f0f@sha256:38e8941107bd19eb30bdde6e478760a22325f38d1f2771dfd1b9af81d74235e7
ARG CILIUM_BPFTOOL_IMAGE=quay.io/cilium/cilium-bpftool:d3093f6aeefef8270306011109be623a7e80ad1b@sha256:2c28c64195dee20ab596d70a59a4597a11058333c6b35a99da32c339dcd7df56
ARG CILIUM_IPROUTE2_IMAGE=quay.io/cilium/cilium-iproute2:f882e3fd516184703eea5ee9b3b915748b5d4ee8@sha256:f22b8aaf01952cf4b2ec959f0b8f4d242b95ce279480fbd73fded606ce0c3fa4
ARG CILIUM_IPTABLES_IMAGE=quay.io/cilium/iptables-20.04:e6f83206c57e606282056903ffd3aab0183bdaed@sha256:7ce0de449d356a5259021dc13f2b00a8bddfbea57a1c91ff8f146d455cace9e5

FROM --platform=$TARGETPLATFORM ${RUBBLE_POLICY_IMAGE} as policy-dist
FROM --platform=$TARGETPLATFORM ${CILIUM_LLVM_IMAGE} as llvm-dist
FROM --platform=$TARGETPLATFORM ${CILIUM_BPFTOOL_IMAGE} as bpftool-dist
FROM --platform=$TARGETPLATFORM ${CILIUM_IPROUTE2_IMAGE} as iproute2-dist
FROM --platform=$TARGETPLATFORM ${CILIUM_IPTABLES_IMAGE} as iptables-dist

FROM --platform=$BUILDPLATFORM golang:1.19.6 as builder
ARG GOPROXY
ARG TARGETOS
ARG TARGETARCH
ENV GOPROXY $GOPROXY
# 改为rubble
#WORKDIR /go/src/github.com/AliyunContainerService/terway/
# COPY go.sum go.mod ./
# RUN go mod download
# COPY . .
# RUN cd cmd/terway && CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -tags default_build \
#     -ldflags \
#     "-X \"github.com/AliyunContainerService/terway/pkg/version.gitCommit=`git rev-parse HEAD`\" \
#     -X \"github.com/AliyunContainerService/terway/pkg/version.buildDate=`date -u +'%Y-%m-%dT%H:%M:%SZ'`\" \
#     -X \"github.com/AliyunContainerService/terway/pkg/version.gitVersion=`git describe --tags --match='v*' --abbrev=14`\" \
#     -X \"github.com/AliyunContainerService/terway/pkg/aliyun.kubernetesAlicloudIdentity=Kubernetes.Alicloud/`git rev-parse --short HEAD 2>/dev/null`\"" -o terwayd .
# RUN cd plugin/terway && CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -tags default_build -o terway .
# RUN cd cmd/terway-cli && CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -tags default_build -o terway-cli .
WORKDIR /go/src/github.com/
COPY rubble rubble
RUN cd rubble && GOOS=linux go build cmd/cni/rubble.go  && GOOS=linux go build cmd/cni-daemon/rubble-daemon.go
############
FROM --platform=$TARGETPLATFORM ${UBUNTU_IMAGE}
RUN apt-get update && apt-get install -y kmod libelf1 libmnl0 iptables nftables kmod curl ipset bash ethtool bridge-utils socat grep findutils jq conntrack iputils-ping && \
    apt-get purge --auto-remove && apt-get clean && rm -rf /var/lib/apt/lists/*

COPY --from=llvm-dist /usr/local/bin/clang /usr/local/bin/llc /bin/
COPY --from=bpftool-dist /usr/local /usr/local
COPY --from=iproute2-dist /usr/local /usr/local
COPY --from=iproute2-dist /usr/lib/libbpf* /usr/lib/
# COPY --from=policy-dist /bin/calico-felix /bin/calico-felix
COPY --from=iptables-dist /iptables /iptables
RUN dpkg -i /iptables/*\.deb && rm -rf /iptables

# 引导cilium的脚本
COPY rubble/image-build/policy/policyinit.sh /bin/
#COPY policy/uninstall_policy.sh /bin/
COPY rubble/image-build/policy/init.sh /bin/

COPY --from=policy-dist /tmp/install/ /
COPY --from=builder /go/src/github.com/rubble/rubble-daemon /usr/bin/
COPY --from=builder /go/src/github.com/rubble/rubble       /opt/cni/bin/
COPY --from=builder /go/src/github.com/rubble/image-build/iptables-wrapper-installer.sh /iptables-wrapper-installer.sh
RUN /iptables-wrapper-installer.sh --no-sanity-check
# 运行rubble
# ENTRYPOINT ["/usr/bin/rubble-daemon"]
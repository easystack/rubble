# how to debug

## build Dockerfile.policy
- cilium构建镜像
```
hub.easystack.cn/cni-devops/cilium-builder:based-681863ee34
# cilium-builder:based-681863ee34是根据官网最新的cilium-builder，更新了go 编译器
# 官方镜像: quay.io/cilium/cilium-builder:681863ee3449dc048f19ac72e63d80d8e7a1b903 
```
- build该Dockerfile时，按照GOPATH的规则，将Cilium项目放到与rubble同级的目录，执行如下
```bash
docker buildx build -t rubble:policy-`date "+%Y%m%d"` \
--build-arg TARGETPLATFORM="linux/amd64" \
--build-arg GOPROXY="https://goproxy.cn,direct" \
-f Dockerfile.policy ../../../cilium
```
## build Dockerfile
```bash
docker buildx build -t hub.easystack.cn/cni-devops/rubble:all-`date "+%Y%m%d"` \
--build-arg TARGETPLATFORM="linux/amd64" \
--build-arg BUILDPLATFORM="linux/amd64" \
--build-arg GOPROXY="https://goproxy.cn,direct" \
--build-arg TARGETOS="linux" \
--build-arg TARGETARCH="amd64" \
-f Dockerfile ../../
```
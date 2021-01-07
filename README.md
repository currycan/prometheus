# prometheus

> prometheus-operator 部署高可用 promethues

## 环境准备

安装 kustomize: [Binaries | SIG CLI](https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/)

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash

# 或者直接 GitHub 下载
VERSION=v3.9.1
https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F${VERSION}/kustomize_${VERSION}_linux_amd64.tar.gz
tar zxvf kustomize_${VERSION}_linux_amd64.tar.gz -C /usr/local/bin
chmod +x /usr/local/bin/kustomize
```

kustomize 语法指导: [kustomization.yaml | SIG CLI](https://kubectl.docs.kubernetes.io/zh/api-reference/kustomization/)

## 安装步骤

[overlays](overlays) 中根据需要进行修改相应的参数, 例如: 服务的暴露方式和 prometheus 的持久化 sc

- 创建 namespace(目前支持在 monitoring namespace )

```bash
kubectl create ns monitoring
kubectl label ns monitoring component=monitorn owner=andrew
```

- 创建 prometheus 采集指标认证密钥

```bash
etcd_cert_path="/etc/kubernetes/pki/etcd"
# etcd_ca="${etcd_cert_path}/ca.pem"
# etcd_key="${etcd_cert_path}/healthcheck-client-key.pem"
# etcd_crt="${etcd_cert_path}/healthcheck-client.pem"
etcd_ca="${etcd_cert_path}/ca.crt"
etcd_key="${etcd_cert_path}/healthcheck-client.key"
etcd_crt="${etcd_cert_path}/healthcheck-client.crt"
kubectl -n monitoring create secret generic etcd-certs --from-file=${etcd_crt} --from-file=${etcd_key} --from-file=${etcd_ca}
```

- 部署应用

```bash
# crd资源安装
kustomize build crds | kubectl apply -f -
# rolebindings 安装
kustomize build rolebindings | kubectl apply -f -
# 核心组件安装
kustomize build . | kubectl apply -f -
```

k8s 高版本中集成 kustumize

```bash
kubectl apply -k crds
kubectl apply -k rolebindings
kubectl apply -k .
```

## 卸载

```bash
kustomize build . | kubectl delete -f -
kustomize build crds | kubectl delete -f -
```

## 遇坑说明

- service monitor 采集不到指标排查: 1. 查看 prometheus 日志; 2. 查看 servicemonitor 资源对应端口是否写对?对应标签选择器是否写对?(service 标签);3. 查看 service 对应 endpoint 是否正常?

- service monitor 删除后需要等待一段时间后才会刷新.

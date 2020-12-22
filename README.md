# prometheus

> prometheus-operator 部署高可用 promethues

## 环境准备

安装 kustomize: [Binaries | SIG CLI](https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/)

```bash
curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

kustomize 语法指导: [kustomization.yaml | SIG CLI](https://kubectl.docs.kubernetes.io/zh/api-reference/kustomization/)

## 安装步骤

[overlays](overlays) 中根据需要进行修改相应的参数, 例如: 服务的暴露方式和 prometheus 的持久化 sc

```bash
# crd资源安装
kustomize build crds | kubectl apply -f -
# rolebindings 安装
kustomize build rolebindings | kubectl apply -f -
# 核心组件安装
kustomize build . | kubectl apply -f -
```

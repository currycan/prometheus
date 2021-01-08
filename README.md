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

- 创建 etcd prometheus 采集指标认证密钥

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

- 创建 adapter 认证密钥

```bash
bash create-adapter-cert.sh
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

## 获取 adapter 指标

- 列出由prometheus提供的自定义指标: `kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .`
- 查看新创建的api群组: `kubectl api-versions  | grep metrics`

  ```bash
  # kubectl api-versions  | grep metrics
  custom.metrics.k8s.io/v1beta1
  custom.metrics.k8s.io/v1beta2
  external.metrics.k8s.io/v1beta1
  metrics.k8s.io/v1beta1

  # kubectl api-versions  | grep autoscaling
  autoscaling/v1
  autoscaling/v2beta1
  autoscaling/v2beta2
  ```

- Pod 的Prometheus 适配器所提供的缺省定制指标: `kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .  |grep "pods/"`

- 举例来说,需要对 java 应用 `jvm_threads_states_threads` 指标 HPA, 查询:

  ```bash
  # kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/demo-jmx/pods/*/jvm_threads_states_threads"
  {"kind":"MetricValueList","apiVersion":"custom.metrics.k8s.io/v1beta1","metadata":{"selfLink":"/apis/custom.metrics.k8s.io/v1beta1/namespaces/demo-jmx/pods/%2A/jvm_threads_states_threads"},"items":[{"describedObject":{"kind":"Pod","namespace":"demo-jmx","name":"demo-app-5d45f58f59-wz5fm","apiVersion":"/v1"},"metricName":"jvm_threads_states_threads","timestamp":"2021-01-07T10:19:41Z","value":"27","selector":null}]}
  ```

## 卸载

```bash
kustomize build . | kubectl delete -f -
kustomize build crds | kubectl delete -f -
```

## 遇坑说明

- service monitor 采集不到指标排查: 1. 查看 prometheus 日志; 2. 查看 servicemonitor 资源对应端口是否写对?对应标签选择器是否写对?(service 标签);3. 查看 service 对应 endpoint 是否正常?

- service monitor 删除后需要等待一段时间后才会刷新.

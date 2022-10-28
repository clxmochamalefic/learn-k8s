# k8s yaml 設定 チートシート

## 各ファイルの名前と役割

おおむねこんな感じではあるが、 `kubectl api-resources` で確認した方が適切

| name | kind | desctiption | note |
|:---:|:---:|:---:|:---:|
| nodes | Node | - |
| namespaces | Namespace | 名前空間 | `kubectl apply` でビルドした時にこいつの関係でビルドがミスることがあるので、先に `apply` するのがよい |
| deployments | Deployment | - |
| services | Service | nodeの定義 | - |
| pods | Pod | Podの定義 | serviceを定義してある場合かつpodを別定義したりしなくて良い場合は作成しなくても良い |
| configmaps | ConfigMap | ConfigMapの定義 | - |
| endpoints | Endpoints | EndPointsの定義 | エンドポイントスライスで使用 |

```bash
nao@Dracaena k8s % kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event

:
:

ingressrouteudps                               traefik.containo.us/v1alpha1           true         IngressRouteUDP
middlewares                                    traefik.containo.us/v1alpha1           true         Middleware
middlewaretcps                                 traefik.containo.us/v1alpha1           true         MiddlewareTCP
serverstransports                              traefik.containo.us/v1alpha1           true         ServersTransport
tlsoptions                                     traefik.containo.us/v1alpha1           true         TLSOption
tlsstores                                      traefik.containo.us/v1alpha1           true         TLSStore
traefikservices                                traefik.containo.us/v1alpha1           true         TraefikService
```

## 記載項目

### 基本項目

基本的なフォーマットはおおむねこんな感じ

```yaml
kind: <kind>
apiVersion: <api_version>
metadata:
    name: <name>
spec:
    // :
    // :
```

#### 例外1. `ConfigMap`

`ConfigMap` のみちょっと違う

<https://kubernetes.io/ja/docs/concepts/configuration/configmap/>

```yaml
kind: <kind>
apiVersion: <api_version>
metadata:
    name: <name>
data:
  // こんな風に書ける、key/value-pair
  default_name: "hogehoge"
  default_pass: "fugafuga"

  // ファイルのように書くことも可能
  game.ini |
    username="hogehoge"
    session=1

  // :
  // :
```

### サブドメイン (subdomain)

`pod` の `spec > subdcomain` で指定可能

e.g.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo # Actually, no port is needed.
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  # subdomainの指定
  subdomain: default-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
```

## ここまでざっと読んだあとに読むと良いもの

### k8sコミュニティによるAPIの説明

<https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md>

`metadata` とか `spec` とか、 `.yaml` に出てくるワードについて説明がされています

### namespace の扱いについて

`namespace` を使った方がいいのはそれとして、ファイル別定義にしたとしても、ビルドは以下のコマンドを使ってやった方がいいかもしれない

```bash
$ kubectl apply -f <yaml_filepath_or_dirpath> --namespace=<my_namespace>
```

となると、以下のスクリプトを実行する形がまるいかも

```build.k8s.sh
#!/bin/bash

while getopts n: OPT
do
  case $OPT in
    "n" ) FLG_NS="TRUE" ; VALUE_NS="$OPTARG" ;;
  esac
done

if [ $# -lt 2 ]; then
  echo "Usage: build.k8s.sh <fs_object_path> [-n <namespace_name>]"
  exit 1
fi

if "${FLG_NS}; then
  kubectl create namespace $1
  kubectl apply -f $1 --namespace=$VALUE_NS
else
  kubectl apply -f $1
fi
```

how2use:

```bash
$ build.k8s.sh ~/test.yaml
$ build.k8s.sh ~/test.yaml -n hogehoge
```

ちなみにべっこで `namespace` を作る時は `kubectl create namespace <my_namespace>`

```bash
nao@Dracaena k8s % kubectl create namespace hogens
namespace/hogens created
nao@Dracaena k8s % kubectl get namespace
NAME              STATUS   AGE
default           Active   91d
kube-system       Active   91d
kube-public       Active   91d
kube-node-lease   Active   91d
hogens            Active   10s
nao@Dracaena k8s %
```

`namespace` に所属できるリソースの一覧は以下でそれぞれ取れる

```bash
# Namespaceに属しているもの
$ kubectl api-resources --namespaced=true

# Namespaceに属していないもの
$ kubectl api-resources --namespaced=false
```

## 用語集

### `kind`

リソースタイプのことを `k8s` では `kind` という

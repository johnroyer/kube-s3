以下步驟會透過 s3-fuse 建立 docker image，並讓 k8s pod 能夠 mount AWS s3 bucket。

## Build docker image

以下將建立 docker image，並將 s3 bucket 掛載到 `/usr/share/nginx/html`。

`git clone` 以後，進入根目錄並執行指令：

```sh
docker build -t kube-s3:latest .
```

----

## 啟動 minikube

```sh
minikube start

# 載入前一步驟 build 出來的 image
minikube image load kube-s3:latest
```

## 啟動測試 pod

供 s3-provider 使用的 secret file 在 `yaml/secret.yaml`，更新 bucket name 以及 tokens。

執行指令：

```sh
kubectl apply -f yaml/secret.yaml
kubectl apply -f yaml/daemonset.yaml
```

若此步驟正確執行，將看到 s3-provider 的 pod 成功運作：

```sh
kubectl get pods -o wide
NAME                READY   STATUS    RESTARTS   AGE
s3-provider-wq5nx   1/1     Running   0          20m
```

接著啟動 nginx pod：

```sh
kubectl apply -f yaml/example_pod.yaml
```

確認啟動成功：

```sh
kubectl get pods -o wide
NAME                READY   STATUS              RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
s3-provider-wq5nx   1/1     Running             0          21m   172.17.0.3     minikube   <none>           <none>
test-pd             0/1     ContainerCreating   0          5s    192.168.49.2   minikube   <none>           <none>
```

透過 curl 連線到 nginx，若 s3-fuse mount 成功，nginx 將會回傳 s3 bucket 中的 `index.html` 內容：

```sh
curl http://192.168.49.2
<html><body><p> Hello World </p></body></html>
```



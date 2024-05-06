# hand_written_kubernetes
つくって、壊して、直して学ぶ kubernetes入門を写経したもの

## 第一章のメモ

コンテナが選ばれるようになった理由

* 仮想マシンよりも高速に起動できるようになった
* マイクロサービスアーキテクチャが採用されるようになり、各サービスを、独立して開発・デプロイ・スケーリングするのにコンテナは適していた
* IMO たしかに、サービスごとに違う言語や環境で作りたいとなったときも構築が楽になりそうだ

おためしで、nginxをコンテナで起動してみる

```
docker run --rm --detach --publish 8080:80 --name web nginx:1.25.3
```

--rm は、コンテナが停止した後に自動的にコンテナを削除する
--detach(-d) は、バックグラウンドでコンテナを起動する
--publish は、ホストのポート（今回は8080）とコンテナのポート（今回は80）をマッピングする

```zsh
> curl "http://localhost:8080"
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

htmlファイルを差し替えたイメージを作ってrun

```
> docker run -d --rm -p 8080:80 --name web nginx-custom:1.0.0
18a0d3c7b483d089143ea4e895c5db904b3830e745bd31f562968a1cba468685
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1 | main !1 ?2 -------- 21:08:54
> curl http://localhost:8080
こんにちはー
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1 | main !1 ?2 -------- 21:09:01
>
```

stop

```
> docker stop web
web
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1 | main !1 ?2 -------- 21:09:30
> docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1 | main !1 ?2 -------- 21:09:34
>
```

最後に、本にあるとおり、goで書いたwebサーバをコンテナで動かした。

## 第二章のメモ

kindを使ってクラスタを作った

```
kind create cluster --image=kindest/node:v1.29.0
```

kindのクラスタを削除する

```
kind delete cluster
```


kubectlのconfig情報は、`~/.kube/config`に保存されている

--kubeconfig フラグや KUBECONFIG 環境変数を利用して設定する ことも可能ということ

## 第四章のメモ

クラスター作ったままでやらないといけないので、kindでクラスターを作る
```
kind create cluster --image=kindest/node:v1.29.0
```

kubectlを使って、podの情報を知ることができる

システムコンポーネントのpodを確認するには

```
kubectl get pod --namespace kube-system
```

デフォルトのnamespaceのnodeを確認するには

```
kubectl get nodes
```

これでクラスタがあるかないかも簡易的に確認できる

```
> kind get clusters
kind
```

デフォルトのnamespaceのPodを確認するには

```
kubectl get pod --namespace default
```

defaultのnamespaceにnginxをデプロイする

```
kubectl apply --filename ./nginx.yaml  --namespace default
```

宣言的にリソースを作成するからapplyを使うのはしっくりくる

Podができているか確認する

```
kubectl get pod --namespace default
```

## 第五章のメモ

まずは、kindでクラスタを作る

```
kind create cluster --image=kindest/node:v1.29.0
```
すでに作られていた場合はエラーが出るようだ。

```
> kind create cluster --image=kindest/node:v1.29.0
ERROR: failed to create cluster: node(s) already exist for a cluster with the name "kind"
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1/hello-server/chapter-04 | main --------- 09:29:18
>
```

Podを作る
```
kubectl apply --filename chapter-04/myapp.yaml
```

適用したのは以下。VScodeの拡張機能で、spec指定をしていないから、リソースを食いつぶすかもしれないと警告が出ていた。

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  containers:
  - name: hello-server
    image: blux2/hello-server:1.0
    ports:
      - containerPort: 8080

```

適用した後のPodの状態を確認する

```
 ~/gh/g/1/hand_written_kubernetes/c/hello-server/chapter-04 | main !1 - kind-kind kube | 09:31:01
> kubectl apply --filename myapp.yaml
pod/myapp created
 ~/gh/g/1/hand_written_kubernetes/c/hello-server/chapter-04 | main !1 ?1
> kubectl get pod --namespace default
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          5s
nginx   1/1     Running   0          24h
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1/hello-server/chapter-04 | main !1 ?1 --- 09:34:09
>
```

IPアドレスやNodeを知りたいときはオプションで`-o wide`をつけるといい --output wideでもいい。

```
> kubectl get pod --output wide --namespace default
NAME    READY   STATUS    RESTARTS   AGE    IP           NODE                 NOMINATED NODE   READINESS GATES
myapp   1/1     Running   0          104s   10.244.0.6   kind-control-plane   <none>           <none>
nginx   1/1     Running   0          24h    10.244.0.5   kind-control-plane   <none>           <none>
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1/hello-server/chapter-04 | main !1 ?1 --- 09:35:47
>
```

yaml形式、json形式で出力することもできる
この場合は、設定されている詳細をすべて出力される

jsonだと、jqか、--output jsonpath='{.items[*].metadata.name}' とかで、必要な情報だけを取り出すことができる

```


Podのログの調べ方

kubectl get pod <Pod名> --v=<ログレベル>

```
> kubectl get pod myapp --v=6
I0506 09:45:46.710592   34939 loader.go:395] Config loaded from file:  /Users/ksanchu/.kube/config
I0506 09:45:46.727990   34939 round_trippers.go:553] GET https://127.0.0.1:51907/api/v1/namespaces/default/pods/myapp 200 OK in 14 milliseconds
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          11m
```

```
> kubectl get pod myapp --v=7
I0506 09:45:16.435319   34791 loader.go:395] Config loaded from file:  /Users/ksanchu/.kube/config
I0506 09:45:16.438089   34791 round_trippers.go:463] GET https://127.0.0.1:51907/api/v1/namespaces/default/pods/myapp
I0506 09:45:16.438101   34791 round_trippers.go:469] Request Headers:
I0506 09:45:16.438108   34791 round_trippers.go:473]     Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json
I0506 09:45:16.438112   34791 round_trippers.go:473]     User-Agent: kubectl/v1.30.0 (darwin/arm64) kubernetes/7c48c2b
I0506 09:45:16.454918   34791 round_trippers.go:574] Response Status: 200 OK in 16 milliseconds
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          11m
```

kubectlよりもリソースの詳細を知りたいとき
kubectl describe

IPやNodeだけではなく、起動時間やマウント先もわかる

kubectl logs <Pod名> でログを取得できる

Pod内にコンテナが複数ある場合は、コンテナ名を-cで指定する

実際Podを立てるときは、Podにはランダムな名前がつく。
いちいちPodの名前を調べるのは大変なので、DeploymentにひもづくPodのログを調べることもできる
ラベル指定もできる


kubenetes1.25(2022/08/31リリース)以降ならば、デバッグ用のサイドカーコンテナを建てられた

```
> kubectl debug --stdin --tty myapp --image=curlimages/curl:8.4.0 --target=hello-server --namespace default -- sh
Targeting container "hello-server". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-4dk6t.
If you don't see a command prompt, try pressing enter.
~ $ curl localhost
curl: (7) Failed to connect to localhost port 80 after 0 ms: Couldn't connect to server
~ $ curl localhost:8080
Hello, world!~ $ ls
~ $ ls -la
total 16
drwxr-sr-x    1 curl_use curl_gro      4096 May  6 01:15 .
drwxr-xr-x    1 root     root          4096 Oct 11  2023 ..
-rw-------    1 curl_use curl_gro        45 May  6 01:15 .ash_history
~ $ ls /
bin            etc            mnt            root           sys
cacert.pem     home           opt            run            tmp
dev            lib            proc           sbin           usr
entrypoint.sh  media          product_uuid   srv            var
~ $ ls /home/
curl_user
~ $
```

copilotに教えてもらったこと
```
kubectl debug --stdin --tty myapp --image=python:3.8-slim --target=my-python-app --namespace default -- sh
```
で対象のコンテナに入って、pip install ptvsdでアプリケーションにimport ptvsdなどをしてデバッグできるようだ。


Podの再起動は、直接該当するコマンドがないため、kubectl delete <リソース名>で削除することで再起動できる。
Deployment単位で順番に再起動したい場合は、kubectl rollout restartらしいが、試すためにどんな引数を渡せばいいのか、わからなかった。。

memo

自動補完設定は、 https://kubernetes.io/docs/reference/kubectl/quick-reference/#kubectl-autocomplete でやり方がわかる。


デバッグの方針

Podから切り分けていく。
Pod->ReplicaSet->Deployment->Service->Ingress


今回はPodを本にあるとおり壊して
kubectl get pod myapp --namespace default
kubectl describe pod myapp --namespace=default
で、Podの状態を確認して、Imageのpullができていないことがわかった。

kubectl edit pod myapp --namespace default
で修正する。

```
> kubectl edit pod myapp --namespace default
pod/myapp edited
 ~/gh/g/1/hand_written_kubernetes/c/hello-server/chapter-05 | main !1 ?4
> kubectl get pod myapp --namespace default
NAME    READY   STATUS    RESTARTS       AGE
myapp   1/1     Running   1 (4m2s ago)   103m
 ~/gh/g/1021ky/hand_written_kubernetes/chapter1/hello-server/chapter-05 | main !1 ?4 --- 11:17:46
>
```

動いていることを確認できた。

お片付け。
ファイル指定で、削除できるようだ
```
> kubectl delete --filename pod-destruction.yaml --namespace default

pod "myapp" deleted
> kubectl get pod myapp --namespace default
Error from server (NotFound): pods "myapp" not found
```


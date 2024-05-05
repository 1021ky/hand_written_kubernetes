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


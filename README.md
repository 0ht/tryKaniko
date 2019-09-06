# Kanikoを試行する

Kanikoは、コンテナーやk8sクラスター内部でコンテナーイメージをビルドしてくれるツール。

Kanikoは、コンテナーイメージ`gcr.io/kaniko-project/executor`として動作して、DockerfileからイメージをビルドしてレジストリーにPUSHしてくれる。

Kanikoを動かすために必要なものは、
- build context(ビルドする場所)
- Kanikoのインスタンス

**build context**としては、以下が利用可能
- GCS Bucket
- S3 Bucket
- Local Directory
- Git Repository

また、Kanikoを動かす方法としては以下が利用可能
- In a Kubernetes cluster
- In gVisor
- In Google Cloud Build
- In Docker

今回は、`Git Repository` と`ローカルのk8sクラスター`を利用して試行してみる。

Kubernetsで動かすのに必要なものは、
1. k8s
2. Kubernetes secret
3. build context
   

1.のk8sは、ローカルのDocker Desktop提供のものを利用

2.は以下の様にして作成
```
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=ohtom \
  --docker-password=XXXXX \
  --docker-email=oh.tomtom@gmail.com
```
実際のsecret作成の実行結果。
```
(⎈ |dev.kops.k8s.local:istio-system)eb82649:transfer eb82649@jp.ibm.com$ kubectl create secret docker-registry regcred \
>   --docker-server=https://index.docker.io/v1/ \
>   --docker-username=ohtom \
>   --docker-password=XXXXX \
>   --docker-email=oh.tomtom@gmail.com
secret/regcred created
```

実行してみる。
```
(⎈ |docker-desktop:default)eb82649:tryKaniko eb82649@jp.ibm.com$ k apply -f kaniko-local-git.yaml 
pod/kaniko-git created
(⎈ |docker-desktop:default)eb82649:tryKaniko eb82649@jp.ibm.com$ k get pods
NAME                                   READY   STATUS              RESTARTS   AGE
echo-hello-world-task-run-pod-b8c449   0/1     Completed           0          15d
kaniko-git                             0/1     ContainerCreating   0          2s
remotecall-git                         0/1     Error               0          12d
test-git-branch-pod-869080             0/2     Completed           0          15d
test-git-ref-pod-d0a14a                0/2     Completed           0          15d
test-git-tag-pod-e24497                0/2     Completed           0          15d

```

結果確認。
```
(⎈ |docker-desktop:default)eb82649:tryKaniko eb82649@jp.ibm.com$ kubectl logs -f kaniko-git
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Compressing objects: 100% (5/5), done.
Total 8 (delta 0), reused 5 (delta 0), pack-reused 0
INFO[0005] Resolved base name ubuntu to ubuntu          
INFO[0005] Resolved base name ubuntu to ubuntu          
INFO[0005] Downloading base image ubuntu                
INFO[0006] Error while retrieving image from cache: getting file info: stat /cache/sha256:ca013ac5c09f9a9f6db8370c1b759a29fe997d64d6591e9a75b71748858f7da0: no such file or directory 
INFO[0006] Downloading base image ubuntu                
INFO[0008] Built cross stage deps: map[]                
INFO[0008] Downloading base image ubuntu                
INFO[0010] Error while retrieving image from cache: getting file info: stat /cache/sha256:ca013ac5c09f9a9f6db8370c1b759a29fe997d64d6591e9a75b71748858f7da0: no such file or directory 
INFO[0010] Downloading base image ubuntu                
INFO[0011] Skipping unpacking as no commands require it. 
INFO[0011] Taking snapshot of full filesystem...        
INFO[0011] CMD ["echo", "Hello World"]                  
2019/08/21 03:07:45 mounted blob: sha256:8e829fe70a46e3ac4334823560e98b257234c23629f19f05460e21a453091e6d
2019/08/21 03:07:45 mounted blob: sha256:6001e1789921cf851f6fb2e5fe05be70f482fe9c2286f66892fe5a3bc404569c
2019/08/21 03:07:45 mounted blob: sha256:251f5509d51d9e4119d4ffb70d4820f8e2d7dc72ad15df3ebd7cd755539e40fd
2019/08/21 03:07:45 mounted blob: sha256:35c102085707f703de2d9eaad8752d6fe1b8f02b5d2149f1d8357c9cc7fb7d0a
2019/08/21 03:07:47 pushed blob: sha256:39a2460b7b09de27404a0220285cfe483e3f139961d9078c976dd9b98ce56309
2019/08/21 03:07:47 index.docker.io/ohtom/trykaniko:git: digest: sha256:2fe7afeb61d7dde31053edaacbac1c857e17f25d465074394637f5450a892f50 size: 911


(⎈ |docker-desktop:default)eb82649:tryKaniko eb82649@jp.ibm.com$ docker pull ohtom/trykaniko:git
git: Pulling from ohtom/trykaniko
35c102085707: Pull complete 
251f5509d51d: Pull complete 
8e829fe70a46: Pull complete 
6001e1789921: Pull complete 
Digest: sha256:2fe7afeb61d7dde31053edaacbac1c857e17f25d465074394637f5450a892f50
Status: Downloaded newer image for ohtom/trykaniko:git
docker.io/ohtom/trykaniko:git
(⎈ |docker-desktop:default)eb82649:tryKaniko eb82649@jp.ibm.com$ docker images
REPOSITORY                                                        TAG                 IMAGE ID            CREATED             SIZE
ohtom/trykaniko                                                   git                 39a2460b7b09        39 minutes ago      64.2MB

(⎈ |docker-desktop:default)eb82649:tryKaniko eb82649@jp.ibm.com$ docker run ohtom/trykaniko:git

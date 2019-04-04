由于墙的关系，使用 docker hub 的 autobuild 功能，曲线拉取谷歌镜像
参考: [kubernetes— 在国内如何巧妙获取kubernetes各镜像？](https://www.jianshu.com/p/11a804080230)

- colbrze430/hyperkube -> gcr.io/google_containers/hyperkube
- colbrze430/pause -> gcr.io/google_containers/pause
- colbrze430/etcd -> gcr.io/google_containers/etcd


拉取镜像后，修改tag：
```
images=(hyperkube:v0.17.0 etcd:2.0.9 pause:0.8.0)  
for image in ${images[@]} ; do
    docker pull colbrze430/$image

    docker tag colbrze430/$image gcr.io/google_containers/$image

    docker rmi colbrze430/$image
done
```

启动 kubernetes
```
docker run --net=host -d gcr.io/google_containers/etcd:2.0.9 /usr/local/bin/etcd --addr=127.0.0.1:4002 --bind-addr=0.0.0.0:4002 --data-dir=/var/etcd/data

docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock  gcr.io/google_containers/hyperkube:v0.17.0 /hyperkube kubelet --api_servers=http://localhost:8081 --v=2 --address=0.0.0.0 --enable_server --hostname_override=127.0.0.1 --config=/etc/kubernetes/manifests

docker run -d --net=host --privileged gcr.io/google_containers/hyperkube:v0.17.0 /hyperkube proxy --master=http://127.0.0.1:8081 --v=2

```
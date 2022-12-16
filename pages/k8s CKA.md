- ## Deployment
	- 指定镜像部署一个 deployment
	- `kubectl create deployment nginx --image=nginx`
	-
	- 使用`--dry-run-client`，验证命令是否正确，不会实际执行
	- `kubectl create deployment XXX --image=nginx --dry-run=client -o yaml`
	-
	- 使用 yaml 文件替换 deployment：(--force 强制替换)
	- `kubectl replace -f first.yaml  --force`
	-
	- 水平拓展
	- `kubectl scale deployment nginx --replicas=3`
	-
- ### expose deployment
	- 暴露 deployment
	- `kubectl expose deployment/nginx`
	-
	- loadbalancer
	- `kubectl expose deployment nginx --type=LoadBalancer`
	- 获得结果
	  ```
	  root@cp:~# kubectl get svc
	  NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
	  kubernetes   ClusterIP      10.96.0.1     <none>        443/TCP        3d23h
	  nginx        LoadBalancer   10.104.4.51   <pending>     80:31528/TCP   3s
	  ```
	- 从外部 IP 访问
	  ```
	  root@cp:~# curl ifconfig.io
	  34.82.40.194
	  root@cp:~# kubectl get svc
	  NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
	  kubernetes   ClusterIP      10.96.0.1     <none>        443/TCP        3d23h
	  nginx        LoadBalancer   10.104.4.51   <pending>     80:31528/TCP   44s
	  root@cp:~# curl http:34.82.40.194:31528
	  curl: (3) URL using bad/illegal format or missing URL
	  root@cp:~# curl http://34.82.40.194:31528
	  <!DOCTYPE html>
	  <html>
	  <head>
	  <title>Welcome to nginx!</title>
	  <style>
	  html { color-scheme: light dark; }
	  body { width: 35em; margin: 0 auto;
	  font-family: Tahoma, Verdana, Arial, sans-serif; }
	  </style>
	  </head>
	  <body>
	  <h1>Welcome to nginx!</h1>
	  <p>If you see this page, the nginx web server is successfully installed and
	  working. Further configuration is required.</p>
	  
	  <p>For online documentation and support please refer to
	  <a href="http://nginx.org/">nginx.org</a>.<br/>
	  Commercial support is available at
	  <a href="http://nginx.com/">nginx.com</a>.</p>
	  
	  <p><em>Thank you for using nginx.</em></p>
	  </body>
	  </html>
	  ```
- ## etcd
	- 备份 etcd
		- 先找出 etcd 的数据目录
		  `grep data-dir /etc/kubernetes/manifests/etcd.yaml`
		- 进入对应容器
		  `kubectl -n kube-system exec -it etcd-<Tab> -- sh`
		- 找出证书的位置
		  `cd /etc/kubernetes/pki/etcd && echo *`
		- etcd 执行健康检查
		  ```
		  kubectl -n kube-system exec -it etcd-cp -- sh \
		  -c "ETCDCTL_API=3 \
		  ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
		  ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
		  ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
		  etcdctl endpoint health"
		  ```
		- 查看 etcd 实例
			- ```
			  kubectl -n kube-system exec -it etcd-cp -- sh -c "ETCDCTL_API=3 \
			  ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
			  ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
			  ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
			  etcdctl --endpoints=https://127.0.0.1:2379 member list"
			  ```
			- `e6d53080e3777327, started, cp, https://10.2.0.2:2380, https://10.2.0.2:2379, false`
		- 执行备份
			- ```
			  kubectl -n kube-system exec -it etcd-cp -- sh -c "ETCDCTL_API=3 \
			  ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
			  ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
			  ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
			  etcdctl --endpoints=https://127.0.0.1:2379 snapshot save /var/lib/etcd/snapshot.db"
			  ```
		- 备份配置文件
			- `mkdir $HOME/backup`
			- `sudo cp /var/lib/etcd/snapshot.db $HOME/backup/snapshot.db-$(date +%m-%d-%y)`
			- `sudo cp /root/kubeadm-config.yaml $HOME/backup/`
			- `sudo cp -r /etc/kubernetes/pki/etcd $HOME/backup/`
		- [恢复 etcd 集群文档](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster)
- ## 集群升级
	- `sudo apt update`
	- `sudo apt-cache madison kubeadm` 查看包可用的版本
	- `sudo apt-mark unhlod kubeadm`
	- 将 cp 节点的 pod 迁移
		- `kubectl drain cp --ignore-daemonsets` 为 CNI保留了 daemonset
	- 查看执行更新的计划
		- `kubeadm upgrade plan`
		- 结果：
		  ```
		  Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
		  COMPONENT   CURRENT       TARGET
		  kubelet     2 x v1.24.1   v1.24.9
		  
		  Upgrade to the latest version in the v1.24 series:
		  
		  COMPONENT                 CURRENT   TARGET
		  kube-apiserver            v1.24.1   v1.24.9
		  kube-controller-manager   v1.24.1   v1.24.9
		  kube-scheduler            v1.24.1   v1.24.9
		  kube-proxy                v1.24.1   v1.24.9
		  CoreDNS                   v1.8.6    v1.9.3
		  etcd                      3.5.3-0   3.5.4-0
		  
		  You can now apply the upgrade by executing the following command:
		  
		  	kubeadm upgrade apply v1.24.9
		  
		  _____________________________________________________________________
		  ```
	- 执行升级
		- `kubeadm upgrade apply v1.25.1`
		- `apt-get install -y kubelet=1.25.1-00 kubectl=1.25.1-00`
	- 让 node 恢复调度
		- `kubectl uncordon cp`
	- ### 更新 worker node
		- worker side：
		- `apt-mark unhold kubeadm`
		- `apt-get update && sudo apt-get install -y kubeadm=1.25.1-00`
		-
		- cp side：
		- `kubectl drain worker-1 --ignore-daemonsets`
		-
		- worker side：
		- `kubeadm upgrade node`
		- `apt-mark unhold kubelet kubectl`
		- `apt-get install -y kubelet=1.25.1-00 kubectl=1.25.1-00`
		- `apt-mark hold kubelet kubectl`
		- `systemctl daemon-reload`
		- `systemctl restart kubelet`
		- `kubectl uncordon worker-1`
- ## 资源限制
	- workload 资源限制
	- namespace 资源限制
- ## 命令清单
	- `kubectl config view`
	- 开启一个临时 pod：`kubectl run -i -t busybox --image=busybox --restart=Never`
	-
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
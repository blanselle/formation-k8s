## Pull an Image from a Private Registry

[Official Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

```shell
kubectl create secret docker-registry gitlab \
  --dry-run=client \
  --output=yaml \
  --docker-server=registry.gitlab.com \
  --docker-username=DOCKER_USER \
  --docker-password=DOCKER_PASSWORD
```


## Copy SQLite database to PVC

```shell
cd symfony-demo
kubectl cp data/database.sqlite POD_ID:/srv/app/data/database.sqlite --no-preserve=true -c php
```


### Example startup probe

```yaml
      containers:
        - name: nginx
          image: registry.gitlab.com/widop/internal/formation-k8s/formation-k8s-eks/symfony_nginx:1.1
          args:
            - /bin/sh
            - -c
            - sleep 16; nginx -g 'daemon off;'
          startupProbe:
            httpGet:
              port: 80
              path: /fr
            initialDelaySeconds: 5
            periodSeconds: 5
```



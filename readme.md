
# Goal

Test the file provider in k8s and check if the file updates are detected on the pod by traefik.

# How to Start the Stack

```bash
k3d create --api-port 6550 --publish 80:80 --publish 8080:8080 
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
k3d i containous/traefik:latest
kubectl apply -f conf/
k3d delete
```

If there is an issue on `kubectl apply -f conf/`, reapply the command.

Change the file/configMap then apply the configuration and wait for a whiale to see if a file provider event occurred.


# File Modification Detection

| File provider | VolumeMounts | Volume    | without remove watcher          | with remove watcher                |
|---------------|--------------|-----------|---------------------------------|----------------------------------- |
| on directory  | mountPath    | configMap | change detected after a while   | change detected after a while      |
| on file       | mountPath    | configMap | change not detected             | detect change but conf not applied |


The remove watcher check if a remove event comes and try to re-add the directory

## notes

Failed to operate with a VolumeMounts subPath like :

```yaml
         volumeMounts:
            - mountPath: /etc/traefik/conf/
              name: traefik-dynamic-configmap
              subPath: conf/dynamic.toml
              readOnly: false
```

Detect remove event but failed to update the configuration:
```bash
time="2020-02-17T10:17:21Z" level=error msg="Could not remove watcher: can't remove non-existent inotify watch for: /etc/traefik/conf/..2020_02_17_10_15_27.984411093" providerName=file
```

# References

* https://github.com/containous/traefik/pull/5037
  * http://thrawn01.org/posts/2016/03/22/howto-write-configmap-enabled-golang-microservices/
  * https://medium.com/@xcoulon/kubernetes-configmap-hot-reload-in-action-with-viper-d413128a1c9a
* https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-volumes-example-nfs-persistent-volume.html
* https://kubernetes.io/docs/concepts/storage/volumes/
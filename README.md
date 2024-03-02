# FluxCD


![](/images/13-image01.png)

Let review the gitrepositories from the flux-system

```sh
$ kubectl get gitrepositories -n flux-system
NAME          URL                                              AGE   READY   STATUS
flux-system   https://github.com/FariusGitHub/flux-infra.git   45m   True    stored artifact for revision 'main@sha1:34fc5af6a2c97775a34faac6f8dc8db64ab78480'
instavote     https://github.com/FariusGitHub/instavote.git    39m   True    stored artifact for revision 'main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc'

```

Looking at instavote gitrepositories, we can see

```sh
$ kubectl describe gitrepositories instavote -n flux-system
Name:         instavote
Namespace:    flux-system
Labels:       <none>
Annotations:  <none>
API Version:  source.toolkit.fluxcd.io/v1
Kind:         GitRepository
Metadata:
  Creation Timestamp:  2024-02-29T00:40:27Z
  Finalizers:
    finalizers.fluxcd.io
  Generation:        1
  Resource Version:  6450
  UID:               cd059dae-a872-4304-9440-474dc2cac322
Spec:
  Interval:  30s
  Ref:
    Branch:  main
  Timeout:   60s
  URL:       https://github.com/FariusGitHub/instavote.git
Status:
  Artifact:
    Digest:            sha256:8adbf7396c02450841eca9c933207d8e8ff0c2092291a36357432330a0abf720
    Last Update Time:  2024-02-29T00:40:29Z
    Path:              gitrepository/flux-system/instavote/746286bc6b3c7fbd1ae0467e231fc009657906bc.tar.gz
    Revision:          main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc
    Size:              62758
    URL:               http://source-controller.flux-system.svc.cluster.local./gitrepository/flux-system/instavote/746286bc6b3c7fbd1ae0467e231fc009657906bc.tar.gz
  Conditions:
    Last Transition Time:  2024-02-29T00:40:29Z
    Message:               stored artifact for revision 'main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc'
    Observed Generation:   1
    Reason:                Succeeded
    Status:                True
    Type:                  Ready
    Last Transition Time:  2024-02-29T00:40:29Z
    Message:               stored artifact for revision 'main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc'
    Observed Generation:   1
    Reason:                Succeeded
    Status:                True
    Type:                  ArtifactInStorage
  Observed Generation:     1
Events:
  Type    Reason                 Age                 From               Message
  ----    ------                 ----                ----               -------
  Normal  NewArtifact            41m                 source-controller  stored artifact for commit 'Update vote-deployment.yaml'
  Normal  GitOperationSucceeded  99s (x80 over 41m)  source-controller  no changes since last reconcilation: observed revision 'main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc'

```

and for the kustomization we can see

```sh
$ kubectl describe kustomization vote-dev -n flux-system
Name:         vote-dev
Namespace:    flux-system
Labels:       <none>
Annotations:  <none>
API Version:  kustomize.toolkit.fluxcd.io/v1
Kind:         Kustomization
Metadata:
  Creation Timestamp:  2024-02-29T00:44:29Z
  Finalizers:
    finalizers.fluxcd.io
  Generation:        1
  Resource Version:  11390
  UID:               27dbbae6-e7a4-4585-9d50-49eee0c4c4f6
Spec:
  Force:     false
  Interval:  1m0s
  Path:      ./deploy/vote
  Prune:     false
  Source Ref:
    Kind:            GitRepository
    Name:            instavote
  Target Namespace:  instavote
Status:
  Conditions:
    Last Transition Time:  2024-02-29T01:26:00Z
    Message:               Applied revision: main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc
    Observed Generation:   1
    Reason:                ReconciliationSucceeded
    Status:                True
    Type:                  Ready
  Inventory:
    Entries:
      Id:                   instavote_vote__Service
      V:                    v1
      Id:                   instavote_vote_apps_Deployment
      V:                    v1
  Last Applied Revision:    main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc
  Last Attempted Revision:  main@sha1:746286bc6b3c7fbd1ae0467e231fc009657906bc
  Observed Generation:      1
Events:
  Type     Reason                Age                From                  Message
  ----     ------                ----               ----                  -------
  Normal  ReconciliationSucceeded  94s (x29 over 29m)  kustomize-controller  (combined from similar events): Reconciliation finished in 679.10647ms, next run in 1m0s
```

## INSTALL kustomize

This link is a good reference to install [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/). 

```sh
# pull the compatible tar file to your system, says I have amd64 architecture on ubuntu
$ wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv5.3.0/kustomize_v5.3.0_linux_amd64.tar.gz

$ tar xzf kustomize_v5.3.0_linux_amd64.tar.gz
$ sudo mv kustomize /usr/local/bin
$ sudo chmod +x /usr/local/bin/kustomize

$ kustomize version
v5.3.0
```
Let's go into a small exercise to create a kustomization.yaml as follow where in the same folder we already have the other yaml files. As soon as autodetect was run a new kustomization.yaml will be created as follow.

```sh
$ ls
  vote-deployment.yaml  vote-service.yaml

$ kustomize create --autodetect

$ tree
.
├── kustomization.yaml
├── vote-deployment.yaml
└── vote-service.yaml

$ cat kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization
  resources:
  - vote-deployment.yaml
  - vote-service.yaml

```

Let's review back the 30s interval we made [earlier](https://github.com/FariusGitHub/GitOpsFluxCD/blob/main/README.md), we can save into a gitrepository yaml file as follow

```sh
$ pwd
.../flux-infra/cluster/dev

$ flux export source git instavote >> instavote-gitrepository.yaml
$ cat instavote-gitrepository.yaml
  ---
  apiVersion: source.toolkit.fluxcd.io/v1
  kind: GitRepository
  metadata:
    name: instavote
    namespace: flux-system
  spec:
    interval: 30s
    ref:
      branch: main
    timeout: 1m0s
    url: https://github.com/FariusGitHub/instavote.git
```
And we can repeat to extract a manifest from vote-dev kustomize object as follow

```sh
$ flux export kustomization vote-dev >> vote-dev-kustomization.yaml
$ cat vote-dev-kustomization.yaml
  ---
  apiVersion: kustomize.toolkit.fluxcd.io/v1
  kind: Kustomization
  metadata:
    name: vote-dev
    namespace: flux-system
  spec:
    interval: 1m0s
    path: ./deploy/vote
    prune: false
    sourceRef:
      kind: GitRepository
      name: instavote
    targetNamespace: instavote
```

## MAKING CHANGES FURTHER
Let's see the existing pods made a while ago
```sh
$ kubectl get pods -n instavote
NAME                    READY   STATUS    RESTARTS        AGE
vote-649dc6575c-2pfmq   1/1     Running   1 (4h52m ago)   43h
vote-649dc6575c-bkw5p   1/1     Running   1 (4h52m ago)   42h
vote-649dc6575c-mjd7k   1/1     Running   1 (4h52m ago)   43h
```
Picking one of the pod, we can see as follow. Look at the end (no events)
```sh
$ kubectl describe pod vote-649dc6575c-2pfmq -n instavote
Name:             vote-649dc6575c-2pfmq
Namespace:        instavote
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.9
Start Time:       Wed, 28 Feb 2024 19:47:35 -0500
Labels:           app=vote
                  pod-template-hash=649dc6575c
Annotations:      <none>
Status:           Running
IP:               10.1.0.25
IPs:
  IP:           10.1.0.25
Controlled By:  ReplicaSet/vote-649dc6575c
Containers:
  vote:
    Container ID:   docker://015c6034851028369060193c74b98e8e9389be5261edcfb47ca1066ce7449a58
    Image:          schoolofdevops/vote:v1
    Image ID:       docker-pullable://schoolofdevops/vote@sha256:0bcdeb55763d0e535f287191e7d000009a4540950aed864bfb0671abaf596e0d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 01 Mar 2024 10:43:22 -0500
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Wed, 28 Feb 2024 19:48:07 -0500
      Finished:     Fri, 01 Mar 2024 10:41:02 -0500
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kpqzs (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-kpqzs:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```
As soon as we couple changes below like adding one more replicas and increase the version, we would see something like below.
![](/images/13-image02.png)

```sh
$ kubectl get pods -n instavote
NAME                    READY   STATUS    RESTARTS   AGE
vote-74b56d9cb4-cctd9   1/1     Running   0          75s
vote-74b56d9cb4-gdmjv   1/1     Running   0          50s
vote-74b56d9cb4-k22jx   1/1     Running   0          57s
vote-74b56d9cb4-wwdfx   1/1     Running   0          73s
```
As we can see the 4 brand new pods are created, and if we go to one we can see the changes

```sh
$ kubectl describe pod vote-74b56d9cb4-wwdfx -n instavote
Name:             vote-74b56d9cb4-wwdfx
Namespace:        instavote
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.9
Start Time:       Fri, 01 Mar 2024 15:45:09 -0500
Labels:           app=vote
                  pod-template-hash=74b56d9cb4
Annotations:      <none>
Status:           Running
IP:               10.1.0.32
IPs:
  IP:           10.1.0.32
Controlled By:  ReplicaSet/vote-74b56d9cb4
Containers:
  vote:
    Container ID:   docker://c7345be2af69a070aa047e3dc0c42e5fe131c371d42069c34d197264d2629997
    Image:          schoolofdevops/vote:v4
    Image ID:       docker-pullable://schoolofdevops/vote@sha256:75706d33afced28f37b1d20233282248f6331c33f204fc1b96d9bdfa7c51c923
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 01 Mar 2024 15:45:23 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r6sxf (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-r6sxf:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m5s  default-scheduler  Successfully assigned instavote/vote-74b56d9cb4-wwdfx to docker-desktop
  Normal  Pulling    2m    kubelet            Pulling image "schoolofdevops/vote:v4"
  Normal  Pulled     114s  kubelet            Successfully pulled image "schoolofdevops/vote:v4" in 1.439s (6.568s including waiting)
  Normal  Created    111s  kubelet            Created container vote
  Normal  Started    110s  kubelet            Started container vote
```

## HEALTH CHECK
The next thing we will try to do is inserting a health check in kustomization.<br>

If we review the kustomization profile now, it would be as follow

```sh
$ flux export kustomization vote-dev
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: vote-dev
  namespace: flux-system
spec:
  interval: 1m0s
  path: ./deploy/vote
  prune: false
  sourceRef:
    kind: GitRepository
    name: instavote
  targetNamespace: instavote
```
Actually this vote-dev kustomization was created through this command
```sh
flux create kustomization vote-dev --source=instavote --path="./deploy/vote" --prune=false --interval=1m --target-namespace=instavote
```
If we tweak the command become
```sh
flux create kustomization vote-dev --source=GitRepository/instavote --path="./deploy/vote" --prune=true --interval=1m --health-check="Deployment/vote.instavote" --namespace=flux-system --target-namespace=instavote
```
And we review the vote-dev kustomization again, we should see the healthcheck spec in now there.
```sh
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: vote-dev
  namespace: flux-system
spec:
  healthChecks:
  - kind: Deployment
    name: vote
    namespace: instavote
  interval: 1m0s
  path: ./deploy/vote
  prune: true
  sourceRef:
    kind: GitRepository
    name: instavote
  targetNamespace: instavote
  timeout: 2m0s
```

Says, if we purposely make a mistake from v1 into vx below, the healthcheck may show some message.
Reverting the vx back into v1, the healthcheck message is now gone. 

![](/images/13-image03.png)

Similiarly we could implement the changes in different folder like below. <br>
In this case we will switch kustomization vote-dev from ./deploy/vote to ./deploy/vote-dev <br>



```sh
$ kubectl get pods -n instavote
  NAME                    READY   STATUS    RESTARTS   AGE
  vote-75f85c979d-bmpq4   1/1     Running   0          16s
  vote-75f85c979d-vdfhn   1/1     Running   0          9s

$ flux export kustomization vote-dev
  ---
  apiVersion: kustomize.toolkit.fluxcd.io/v1
  kind: Kustomization
  metadata:
    name: vote-dev
    namespace: flux-system
  spec:
    healthChecks:
    - kind: Deployment
      name: vote
      namespace: instavote
    interval: 1m0s
    path: ./deploy/vote
    prune: true
    sourceRef:
      kind: GitRepository
      name: instavote
    targetNamespace: instavote
    timeout: 2m0s

$ flux delete kustomization vote-dev
  Are you sure you want to delete this kustomization: y
  ► deleting kustomization vote-dev in flux-system namespace
  ✔ kustomization deleted

$ flux export kustomization vote-dev
  ✗ kustomizations.kustomize.toolkit.fluxcd.io "vote-dev" not found

$ flux create kustomization vote-dev --source=GitRepository/instavote --path="./deploy/vote-dev" --prune=true --interval=1m --health-check="Deployment/vote.instavote" --namespace=flux-system --target-namespace=instavote
  ✚ generating Kustomization
  ► applying Kustomization
  ✔ Kustomization created
  ◎ waiting for Kustomization reconciliation
  ✔ Kustomization vote-dev is ready
  ✔ applied revision main@sha1:b76fd25c2ad0ec6f2a17a6fdb6e40d7d319bda99

$ flux export kustomization vote-dev
  ---
  apiVersion: kustomize.toolkit.fluxcd.io/v1
  kind: Kustomization
  metadata:
    name: vote-dev
    namespace: flux-system
  spec:
    healthChecks:
    - kind: Deployment
      name: vote
      namespace: instavote
    interval: 1m0s
    path: ./deploy/vote-dev
    prune: true
    sourceRef:
      kind: GitRepository
      name: instavote
    targetNamespace: instavote
    timeout: 2m0s

$ kubectl get pods -n instavote
NAME                    READY   STATUS    RESTARTS   AGE
vote-74b56d9cb4-fv7qj   1/1     Running   0          10s
vote-74b56d9cb4-pqprq   1/1     Running   0          23s
vote-74b56d9cb4-tb68f   1/1     Running   0          12s
vote-74b56d9cb4-zcsfz   1/1     Running   0          22s
```

![](/images/13-image04.png)
# Smoke Test

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

## Data Encryption

In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Create a generic secret:

```bash
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

Print a hexdump of the `kubernetes-the-hard-way` secret stored in etcd:

```bash
ssh root@server \
    'etcdctl get /registry/secrets/default/kubernetes-the-hard-way | hexdump -C'
```

```text
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a c9 d3 15 93 8f bd 13  |:v1:key1:.......|
00000050  38 d0 7b fb 13 e1 b7 bb  2d 12 fe 4e 60 d9 78 53  |8.{.....-..N`.xS|
00000060  43 1c 69 b0 17 f7 6c f0  fd 87 46 be e1 bb c5 ed  |C.i...l...F.....|
00000070  3c b3 70 b2 be 17 16 2f  09 2f 22 f1 52 82 4f de  |<.p...././".R.O.|
00000080  16 63 14 84 b4 04 8e 27  11 12 ce d6 7c a0 67 b5  |.c.....'....|.g.|
00000090  c3 d0 ee cf 40 dc ee 0f  e6 7a a7 bd 25 23 1c 5e  |....@....z..%#.^|
000000a0  d3 66 35 e3 99 44 8a 8c  57 8f a8 62 49 61 53 d6  |.f5..D..W..bIaS.|
000000b0  17 54 e1 62 6b c8 f2 e8  a7 52 7b 6d 48 75 af 1b  |.T.bk....R{mHu..|
000000c0  a5 3b 1b 1d 3b 59 ab 7e  15 77 64 88 2a 04 86 63  |.;..;Y.~.wd.*..c|
000000d0  c9 9f a6 6e 2a 9c e8 50  31 7b d7 25 29 27 90 8c  |...n*..P1{.%)'..|
000000e0  7c fa fe e3 56 90 ad 87  48 64 b5 6b 77 14 29 2c  ||...V...Hd.kw.),|
000000f0  94 51 98 9b ab f9 1e 15  ff da 76 ad ac 7b 17 72  |.Q........v..{.r|
00000100  ec 89 bb 08 cc 4d 6d c7  1c da 26 5e e1 53 d6 79  |.....Mm...&^.S.y|
00000110  b6 c8 cd 82 2a 4e 2a 76  3f 17 ff 12 e8 cb e8 d5  |....*N*v?.......|
00000120  79 25 cb 99 df d5 e3 1a  f2 6f bb 21 21 ba 06 05  |y%.......o.!!...|
00000130  39 ba 35 0f 7d 31 1b 7f  0c 6f 68 b4 e4 65 fe 91  |9.5.}1...oh..e..|
00000140  bf 2b 9c b2 0c 8f 60 55  8c 1b a4 4f 44 ef a8 1f  |.+....`U...OD...|
00000150  34 75 84 8b 09 2b ce 99  c9 0a                    |4u...+....|
0000015a
```

The etcd key should be prefixed with `k8s:enc:aescbc:v1:key1`, which indicates the `aescbc` provider was used to encrypt the data with the `key1` encryption key.

## Deployments

In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Create a deployment for the [nginx](https://nginx.org/en/) web server:

```bash
kubectl create deployment nginx \
  --image=nginx:latest
```

List the pod created by the `nginx` deployment:

```bash
kubectl get pods -l app=nginx
```

```bash
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56fcf95486-c8dnx   1/1     Running   0          8s
```

### Port Forwarding

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Retrieve the full name of the `nginx` pod:

```bash
POD_NAME=$(kubectl get pods -l app=nginx \
  -o jsonpath="{.items[0].metadata.name}")
```

Forward port `8080` on your local machine to port `80` of the `nginx` pod:

```bash
kubectl port-forward $POD_NAME 8080:80
```

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```bash
curl --head http://127.0.0.1:8080
```

```text
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Sun, 29 Oct 2023 01:44:32 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
Connection: keep-alive
ETag: "6537cac7-267"
Accept-Ranges: bytes

```

Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Print the `nginx` pod logs:

```bash
kubectl logs $POD_NAME
```

```text
...
127.0.0.1 - - [01/Nov/2023:06:10:17 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.88.1" "-"
```

### Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```bash
kubectl exec -ti $POD_NAME -- nginx -v
```

```text
nginx version: nginx/1.25.3
```

## Services

In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the `nginx` deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service:

```bash
kubectl expose deployment nginx \
  --port 80 --type NodePort
```

> The LoadBalancer service type can not be used because your cluster is not configured with [cloud provider integration](https://kubernetes.io/docs/getting-started-guides/scratch/#cloud-provider). Setting up cloud provider integration is out of scope for this tutorial.

Retrieve the node port assigned to the `nginx` service:

```bash
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Make an HTTP request using the IP address and the `nginx` node port:

```bash
curl -I http://node-0:${NODE_PORT}
```

```text
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Sun, 29 Oct 2023 05:11:15 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
Connection: keep-alive
ETag: "6537cac7-267"
Accept-Ranges: bytes
```

Next: [Cleaning Up](13-cleanup.md)

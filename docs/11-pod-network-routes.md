# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

## The Routing Table

In this section you will gather the information required to create routes in the `kubernetes-the-hard-way` VPC network.

Print the internal IP address and Pod CIDR range for each worker instance:

```bash
{
  SERVER_IP=$(grep server machines.txt | cut -d " " -f 1)
  NODE_0_IP=$(grep node-0 machines.txt | cut -d " " -f 1)
  NODE_0_SUBNET=$(grep node-0 machines.txt | cut -d " " -f 4)
  NODE_1_IP=$(grep node-1 machines.txt | cut -d " " -f 1)
  NODE_1_SUBNET=$(grep node-1 machines.txt | cut -d " " -f 4)
}
```

```bash
ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```
Above Command Output:

```text
ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
Pseudo-terminal will not be allocated because stdin is not a terminal.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Thu Nov  7 06:15:56 PM UTC 2024

  System load:  0.0                Processes:             109
  Usage of /:   45.0% of 13.16GB   Users logged in:       1
  Memory usage: 44%                IPv4 address for eth0: 192.168.0.122
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

61 updates can be applied immediately.
13 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

New release '24.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

```
The message "Pseudo-terminal will not be allocated because stdin is not a terminal" appears when SSH detects that it’s not in an interactive terminal. To resolve this, add the -t option to your SSH command to force the allocation of a pseudo-terminal. Here’s how you can modify your command:

```bash
ssh -t root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF

```
Result of above command:
```text
ssh -t root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
Pseudo-terminal will not be allocated because stdin is not a terminal.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Thu Nov  7 06:19:36 PM UTC 2024

  System load:  0.0166015625       Processes:             110
  Usage of /:   45.0% of 13.16GB   Users logged in:       1
  Memory usage: 44%                IPv4 address for eth0: 192.168.0.122
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

61 updates can be applied immediately.
13 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

New release '24.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


RTNETLINK answers: File exists
RTNETLINK answers: File exists
```

The RTNETLINK answers: File exists message indicates that the routes you're trying to add are already present in the routing table on the remote server. To avoid this error, you can first check if the route exists before attempting to add it. Here’s an updated script to handle this:

```bash
ssh -t root@server <<EOF
  if ! ip route show | grep -q "${NODE_0_SUBNET}"; then
    ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  fi
  if ! ip route show | grep -q "${NODE_1_SUBNET}"; then
    ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
  fi
EOF

```
Result of above command:
```text
ssh -t root@server <<EOF
  if ! ip route show | grep -q "${NODE_0_SUBNET}"; then
    ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  fi
  if ! ip route show | grep -q "${NODE_1_SUBNET}"; then
    ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
  fi
EOF
Pseudo-terminal will not be allocated because stdin is not a terminal.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Thu Nov  7 06:21:54 PM UTC 2024

  System load:  0.1474609375       Processes:             111
  Usage of /:   45.0% of 13.16GB   Users logged in:       1
  Memory usage: 44%                IPv4 address for eth0: 192.168.0.122
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

61 updates can be applied immediately.
13 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

New release '24.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.
```
It seems the Pseudo-terminal will not be allocated because stdin is not a terminal warning is still appearing. This usually happens when the SSH session doesn’t recognize standard input redirection (<<EOF).

To address this, you can try a different approach by specifying the command directly instead of using a heredoc. Here’s how you can structure it:

```bash
ssh -t root@server "
  if ! ip route show | grep -q '${NODE_0_SUBNET}'; then
    ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  fi
  if ! ip route show | grep -q '${NODE_1_SUBNET}'; then
    ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
  fi
"
```
Result:
```text
ssh -t root@server "
  if ! ip route show | grep -q '${NODE_0_SUBNET}'; then
    ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  fi
  if ! ip route show | grep -q '${NODE_1_SUBNET}'; then
    ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
  fi
"
Connection to server closed.
```
he message Connection to server closed indicates that the SSH connection was established, the commands ran, and then the connection was closed as expected. This likely means your commands executed successfully.

To verify that the routes were added, you can reconnect and check the routing table manually:
```bash
ssh root@server "ip route show"
```
Output:
```text
default via 192.168.0.1 dev eth0 proto static
10.200.0.0/24 via 192.168.0.123 dev eth0
10.200.1.0/24 via 192.168.0.124 dev eth0
192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.122

```
At worker node (node-0), add route of another worker node (node-1)

```bash
ssh -t root@node-0 "
  if ! ip route show | grep -q '${NODE_1_SUBNET}'; then
    ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
  fi"

```
At worker node (node-1), add route of another worker node (node-0)

```bash
ssh -t root@node-1 "
  if ! ip route show | grep -q '${NODE_0_SUBNET}'; then
    ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  fi
  "
```


## Verification 

```bash
ssh root@server ip route
```

```text
default via 192.168.0.1 dev eth0 proto static
10.200.0.0/24 via 192.168.0.123 dev eth0
10.200.1.0/24 via 192.168.0.124 dev eth0
192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.122
```

```bash
ssh root@node-0 ip route
```

```text
default via 192.168.0.1 dev eth0 proto static
10.200.1.0/24 via 192.168.0.124 dev eth0
192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.123
```

```bash
ssh root@node-1 ip route
```

```text
default via 192.168.0.1 dev eth0 proto static
10.200.0.0/24 via 192.168.0.123 dev eth0
192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.124
```


Next: [Smoke Test](12-smoke-test.md)

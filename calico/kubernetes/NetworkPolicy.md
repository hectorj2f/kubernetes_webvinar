<!--- master only -->
> ![warning](../../images/warning.png) This document applies to the HEAD of the calico-docker source tree.
>
> View the calico-docker documentation for the latest release [here](https://github.com/projectcalico/calico-containers/blob/v0.19.0/README.md).
<!--- else
> You are viewing the calico-docker documentation for release **release**.
<!--- end of master only -->

# Calico Policy for Kubernetes
Calico supports the v1alpha1 network policy API for Kubernetes.
> *Note*: The Kubernetes network policy API is currently in alpha and is subject to change. Calico support for this API is in beta and so is also subject to change.

## Prerequisites
* A Kubernetes v1.1+ deployment using the [Calico CNI plugin v1.1](https://github.com/projectcalico/calico-cni/releases/latest) or greater.
* You must be running the `calico/node:v0.17.0-k8s-policy` image on each Kubernetes node.  This is a beta pre-release image which supports this policy.
* You must be using the iptables kube-proxy in your deployment. All of the Calico getting started guides configure the kube-proxy in this way.
* You must have enabled `ThirdPartyResource` objects in your Kubernetes apiserver, as described [here](https://github.com/caseydavenport/kubernetes/blob/network-policy/docs/admin/network-policy.md#enabling-network-policy).

## Behavior
Calico implements the behavior of the Kubernetes [v1alpha1 network policy API](https://github.com/caseydavenport/kubernetes/blob/network-policy/docs/admin/network-policy.md#network-policy-in-kubernetes).

## Enabling v1alpha1 policy support

To enable annotation-based policy in the Calico CNI plugin, add the `policy` section to your CNI network config file as shown - you will need to make this change on each Kubernetes worker node in your cluster (any node that allows scheduling of pods).  The CNI network configuration file can usually be found in the `/etc/cni/net.d/` directory.
```
$ cat /etc/cni/net.d/10-calico.conf
{
    "name": "calico-k8s-network",
    "type": "calico",
    "etcd_authority": "<ETCD_IP:ETCD_PORT>",
    "log_level": "info",
    "ipam": {
        "type": "calico-ipam"
    },
    "policy": {
        "type": "default-deny-inbound",
    }
}
```

This will disable inbound traffic to pods by default, delegating policy configuration to the [Calico policy agent](https://github.com/projectcalico/k8s-policy).

Once you have modified the network configuration file as shown above, you will need to restart the `kubelet` to pick up the changes.

>Example for `systemd`:
```
sudo systemctl restart kubelet
```

## Running the Calico policy agent
In order to use the Kubernetes v1alpha1 network policy API, you must run the
Calico Kubernetes policy agent.  The policy agent runs as a pod on your
Kubernetes cluster.  It reads policy information from the Kubernetes API and
configures Calico appropriately.

To run the Calico Kubernetes policy agent:

1. Download the policy services manifest file.
```
wget https://raw.githubusercontent.com/projectcalico/k8s-policy/master/examples/calico-policy-services.yaml
```

2. Replace `<ETCD_HOST>` and `<ETCD_PORT>` with the correct configuration to access your etcd cluster.

3. Create the manifest file using `kubectl`.
```
kubectl create -f calico-policy-services.yaml
```

After a few moments, you should see the calico-policy-agent pod running in the `calico-system` namespace.
```
$ ./kubectl get pods --namespace=calico-system
NAME                        READY     STATUS    RESTARTS   AGE
calico-policy-agent-axg63   1/1       Running   0          1m
```

## Next Steps
- Install the [policy tool](https://github.com/projectcalico/k8s-policy/blob/master/policy_tool/README.md) for easy management of NetworkPolicy objects.

- Once you have enabled network policy on your cluster and configured Calico to use the Kubernetes network
policy API, you can deploy our [Kubernetes policy example application](stars-demo/README.md).

[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/cni/kubernetes/NetworkPolicy.md?pixel)](https://github.com/igrigorik/ga-beacon)

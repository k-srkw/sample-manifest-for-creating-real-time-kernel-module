Please refer OpenShift document about the procedure non real-time kernel https://docs.openshift.com/container-platform/4.8/scalability_and_performance/psap-driver-toolkit.html

# Prerequisites

- An OpenShift Container Platform cluster
- Install the OpenShift CLI (oc).
- Log in as a user with cluster-admin privileges.

# Procedure

Create a namespace.

```bash
$ oc new-project simple-rt-kmod-demo
```

Substitute the correct driver toolkit image for the OpenShift Container Platform version you are running in place of “DRIVER_TOOLKIT_IMAGE” with the following commands.

```bash
$ OCP_VERSION=$(oc get clusterversion/version -ojsonpath={.status.desired.version})
$ DRIVER_TOOLKIT_IMAGE=$(oc adm release info $OCP_VERSION --image-for=driver-toolkit)
$ sed "s#DRIVER_TOOLKIT_IMAGE#${DRIVER_TOOLKIT_IMAGE}#" 0000-rt-buildconfig.yaml.template > 0000-rt-buildconfig.yaml
```

Create the image stream and build config with

```bash
$ oc create -f 0000-rt-buildconfig.yaml
```

After the builder pod completes successfully, deploy the driver container image as a DaemonSet.

```bash
$ oc create -f 1000-rt-drivercontainer.yaml
```

After the pods are running on the worker nodes, verify that the simple_kmod kernel module is loaded successfully on the host machines with lsmod.

```bash
$ oc get pod -n simple-rt-kmod-demo
NAME                                    READY   STATUS      RESTARTS   AGE
simple-rt-kmod-driver-build-1-build     0/1     Completed   0          2m59s
simple-rt-kmod-driver-container-47pgg   1/1     Running     0          11s
simple-rt-kmod-driver-container-kqqck   1/1     Running     0          11s
simple-rt-kmod-driver-container-vwq4z   1/1     Running     0          11s

$ oc exec -it pod/simple-rt-kmod-driver-container-47pgg -- lsmod | grep simple
simple_procfs_kmod     16384  0
simple_kmod            16384  0
```

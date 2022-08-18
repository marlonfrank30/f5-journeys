# f5-journeys installation using vanila kubernetes 

- [OpenShift 4 Bare Metal Install - User Provisioned Infrastructure (UPI)](#openshift-4-bare-metal-install---user-provisioned-infrastructure-upi)
  - [Architecture Diagram](#architecture-diagram)
  - [Download Software](#download-software)
  - [Prepare the 'Bare Metal' environment](#prepare-the-bare-metal-environment)
  - [Configure Environmental Services](#configure-environmental-services)
  - [Generate and host install files](#generate-and-host-install-files)
  - [Deploy OpenShift](#deploy-openshift)
  - [Monitor the Bootstrap Process](#monitor-the-bootstrap-process)
  - [Remove the Bootstrap Node](#remove-the-bootstrap-node)
  - [Wait for installation to complete](#wait-for-installation-to-complete)
  - [Join Worker Nodes](#join-worker-nodes)
  - [Configure storage for the Image Registry](#configure-storage-for-the-image-registry)
  - [Create the first Admin user](#create-the-first-admin-user)
  - [Access the OpenShift Console](#access-the-openshift-console)
  - [Troubleshooting](#troubleshooting)

Read the prerequesites for your linux platform at https://openebs.io/docs/user-guides/prerequisites#ubuntu

## iSCSI services
1) Verify iSCSI services are configured for Ubuntu. If iSCSI initiator is already installed on your node, check that the initiator name is configured and iSCSI service is running using the following commands.

```
sudo cat /etc/iscsi/initiatorname.iscsi
systemctl status iscsid 
```

If the service status is shown as Inactive , then you may have to enable and start iscsid service using the following command.

```
sudo systemctl enable --now iscsid
```

Note: The following is the expected output.

systemctl status iscsid
● iscsid.service - iSCSI initiator daemon (iscsid)
   Loaded: loaded (/lib/systemd/system/iscsid.service; disabled; vendor preset: enabled)
   Active: **active** (running) since Mon 2019-02-18 11:00:07 UTC; 1min 51s ago
     Docs: man:iscsid(8)
  Process: 11185 ExecStart=/sbin/iscsid (code=exited, status=0/SUCCESS)
  Process: 11170 ExecStartPre=/lib/open-iscsi/startup-checks.sh (code=exited, status=0/SUCCESS)
 Main PID: 11187 (iscsid)
    Tasks: 2 (limit: 4915)
   CGroup: /system.slice/iscsid.service
           ├─11186 /sbin/iscsid
           └─11187 /sbin/iscsid

2) Installing iSCSI tools 
If iSCSI initiator is not installed on your node, install open-iscsi packages using the following commands.

```
sudo apt-get update
sudo apt-get install open-iscsi
sudo systemctl enable --now iscsid
```

3) install openebs for ubuntu in order to define your storageclass

```
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
```

   to make it a default storageclass in your cluster issue the following command

```
oc patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

   to confirm your changes issue the following command 

```
oc get storageclass -n openebs
```

4) apply the yaml file for the pvc storage class referenced for the pvc local-hostpath-pvc.yaml available as a reference at https://openebs.io/docs/user-guides/localpv-hostpath. the repo has this file already modified with all it is needed.

```
oc apply -f local-hostpath-pvc.yaml
```

***Note: this is already done but for your knowledge, I had to download the f5-journeys repo from github located at https://github.com/f5devcentral/f5-journeys.git and then edit the journeys default deployment yaml file to included the following specs from the example app on item number 5 above. This is already done in the /config files already***

```
spec:
  volumes:
  - name: local-storage
    persistentVolumeClaim:
      claimName: local-hostpath-pvc
```

5) install kompose via a deb package  that is released for compose. Download latest package in the assets in github releases.

```
wget https://github.com/kubernetes/kompose/releases/download/v1.26.1/kompose_1.26.1_amd64.deb # Replace 1.26.1 with latest tag
sudo apt install ./kompose_1.26.1_amd64.deb
```

6) convert the docker container files into kubernetes compliant yaml files using Kompose

```
cd f5-journeys/config
kompose convert -f docker-compose.yaml
kubectl apply -f .
kubectl get po
```

7) apply all the other yaml files inside the config directory and check for the pods and services 
```
kubectl apply -f *
kubect get pods -A
kubect get svc 
```

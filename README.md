# Install wordpress on a CIFS share

## pre-requirements 

- a CIFS share version 2.1
- an IKS cluster with at least 1 available Loadbalancer IP (so you need to have specified a count of atleast 2 LB's)

## Install SMB-CSI driver

Add the repository  
`helm repo add csi-driver-smb https://raw.githubusercontent.com/kubernetes-csi/csi-driver-smb/master/charts`

Install the SMB-CSI  
`helm install csi-driver-smb csi-driver-smb/csi-driver-smb --namespace kube-system`

## Create secrets

Create a secret to access your share, please use the correct username, password and domain.  
`kubectl create secret generic smbcreds --from-literal username=baelen --from-literal password="YOURPASS" --from-literal domain=CISCO`

create a password for your MySQL deployment  
`kubectl create secret generic mysql-pass  --from-literal password="Cisco1234" `

## Install the application

pull down the kubernetes manifests  
`git clone https://github.com/baelen-git/mysql-wordpress-pd`

modfiy the dr-volumes.yaml to point to your CIFS share
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-pv-2
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  mountOptions:
    - dir_mode=0777
    - file_mode=0777
    - vers=2.1
  csi:
    driver: smb.csi.k8s.io
    readOnly: false
    volumeHandle: wordpress-pv-2  
    volumeAttributes:
      source: "//gffa-dc-1/share" # <- PUT YOUR SHARE HERE
    nodeStageSecretRef:
      name: smbcreds
      namespace: default
```

Install the application
```
kubectl create -f dr-volumes.yaml
kubectl create -f mysql-deployment.yaml
kubectl create -f wordpress-deployment.yaml
```


## Login to the application

Check if all pods are running
`kubectl get pod`

get the loadbalancer IP from the wordpress service
`kubectl get svc`

go to the IP of your loadbalancer server for wordpress in your browser.


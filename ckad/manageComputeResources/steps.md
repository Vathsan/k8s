Create a new namespace
`kubectl create namespace resource-test`

Enable the admission controller
`sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml`
`--enable-admission-plugins=ResourceQuota`
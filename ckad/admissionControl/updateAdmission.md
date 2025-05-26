Change the API Server configuration to enable the NamespaceAutoProvision admission controller.
`sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml`

Locate the --enable-admission-plugins flag, and add NamespaceAutoProvision to the list.
`--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision`
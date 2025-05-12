# Installing Helm
Setup the Helm GPG key and package repository.
```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
```

Install the Helm package.
```
sudo apt-get update
sudo apt-get install -y helm
```

Verify the installation.
`helm version`

# Using Helm

Add a Helm chart repository.
`helm repo add bitnami https://charts.bitnami.com/bitnami`

Update the repository.
`helm repo update`

View a list of charts available in the repository.
`helm search repo bitnami`

Create a Namespace.
`kubectl create namespace dokuwiki`

Install a chart.
`helm install --set persistence.enabled=false -n dokuwiki dokuwiki bitnami/dokuwiki`

View some of the objects created by the Helm install.
`kubectl get pods -n dokuwiki`
`kubectl get deployments -n dokuwiki`
`kubectl get svc -n dokuwiki`

Uninstall the release and delete the Namespace to clean up.
`helm uninstall -n dokuwiki dokuwiki`
`kubectl delete namespace dokuwiki`
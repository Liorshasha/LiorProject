First connect to https://shell.cloud.google.com/
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm install jenkins jenkinsci/jenkins
helm install jenkins jenkinsci/jenkins -f values.yaml
kubectl get pods --namespace default -l "app.kubernetes.io/instance=jenkins"
helm status jenkins
kubectl exec --namespace default -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo ---> Save the password !
kubectl --namespace default port-forward svc/jenkins 8080:8080
----------------
 Access via <NODE-IP>:<NODE-PORT>
 User : admin
 After login to Jenkins ..
 Add this addons : Blue Ocean, Pipline: Stage View, Kubernetes CLI

 Add cred before runnig pipeline (Docker hub + Github)




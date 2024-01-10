Install the Linux AWS CLI (v2):

More instructions are here: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

You should now have an `aws` command available.


Next install the sso config.  The simplest approach is to do this via configuration file, although you can also have the cli tool create the configuration with `aws configure sso`.

The file (create in $HOME/.aws/ as config):

```
[sso-session pacioos]
sso_start_url = https://d-9067838cdc.awsapps.com/start
sso_region = us-east-1
sso_registration_scopes = sso:account:access

[profile pacioos-admin]
sso_session = pacioos
sso_account_id = 877956244955
sso_role_name = AdministratorAccess
region = us-west-2
output = json
sso_start_url = https://d-9067838cdc.awsapps.com/start
sso_region = us-east-1
sso_registration_scopes = sso:account:access

[profile pacioos-power]
sso_session = pacioos
sso_account_id = 877956244955
sso_role_name = PowerUserAccess
region = us-west-2
output = json
sso_start_url = https://d-9067838cdc.awsapps.com/start
sso_region = us-east-1
sso_registration_scopes = sso:account:access
```

The recommended approach to logging in with your credentials and establishing your session is as follows:

```
export AWS_PROFILE=pacioos-power
export AWS_REGION=us-west-2

aws sso login
```

This should either point to a web address along with a connection token, or a web page will open with the associated token already incorporated.  Follow the web prompts to complete the authentication (using the user-name/password you established when you registered your account with AWS).

If all of the authentication completes correctly, you should be able to run a simple AWS command like:

```
aws s3 ls
```

Take note of the `start` URL provided by the `aws sso login` command, as this is the quickest way to log into the AWS web console in case there are operations to be done manually in the UI as opposed to using the CLI.

AWS services that may be of interest:

  S3
  EC2
  Route53
  CodeBuilder
## Kubernetes and Helm clients and setup

As the goal is primarily to get to being able to work with, and manipulate the Kubernetes services, 

Install the clients:

### Kubectl

From: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### Helm

From:  https://helm.sh/docs/intro/install/

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Kubectl configuration

We need to establish the Kubectl configuration for the target cluster.  If you’ve already done the AWS sso login (`aws sso login`) then the following commands should be functional:
```
aws eks list-clusters
export CLUSTER-NAME={cluster-name-from-previous-command}
```
Will show the clusters currently available.
```
aws eks update-kubeconfig —-name ${CLUSTER-NAME}
```
Validate that kubectl now works:
```
kubectl get nodes
```
Since we’re often not working in the `default` namespace, we likely want to reset our service to focus on the target namespace by default:
```
K8S_CTX=kubectl config get-contexts | awk '/'pacioos-test'/ {print $2}
kubectl get ns # to list namespaces
kubectl create ns {my-namespace-name}
kubectl config set-contexts $K8S_CTX —namespace {my-namespac—name}
```

We can also check the status of Helm at this point:
```
helm ls
helm repo list
helm repo add wp https://kumulustech.github.io/wordpress-helm-s3
helm repo add ir https://kumulustech.github.io/ingressroute-helm
helm repo add gs https://kumulustech.github.io/geoserver-helm   
helm repo add th https://kumulustech.github.io/thredds-helm     
helm repo add er https://kumulustech.github.io/erddap-helm
```
Deploy the threads app via helm:
```
hclm install erddap er/erddap
```
### Kubectl commands

Here are some useful Kubectl commands:

List running pods:
```
kubectl get pods
kubectl get pods {pod name} -o yaml
kubectl describe pods
kubectl describe pod {pod name} -o yaml
```
Get logs from a pod:
```
kubectl logs {pod-name}
kubectl logs -f {pod-name} -c {container in pod} # -f for follow, -c for container/sidecar/initContainer
```
Exec into Pod
```
kubectl exec -i -t {pod-name} -c {container in pod} —- /bin/sh #or any other pod specific command after the —-
```

### Other Useful tools

#### k9s

k9s is a terminal “norton commander/midnight commander” style management interface for a k8s instance.
https://k9scli.io/
#### kubectx/kubens
A tool for re-naming long context names to shorter, more manageable names.  kubens is a command that is from the same kubectx project, but makes it much more straightforward for manipulating the default namespace
https://kubectx.dev
#### stern
Tail a pods logs without having to find the replicaset-podhash of the current active name.
https://github.com/stern/stern


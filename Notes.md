------------------- EKS-CLUSTER-CREATATION ----------------------


------------------- Java Installation---------------------------

````
sudo apt-get update
sudo apt install openjdk-11-jre-headless
````

-------------------- Install and Setup Jenkins -------------------

````
sudo vi /etc/sudoers
````
````
jenkins ALL=(ALL) NOPASSWD: ALL
````

---------------------- Maven Installation -------------------------



---------------------- Docker Installation -------------------------


````
sudo usermod -aG docker jenkins
sudo chmod 666 /var/run/docker.sock
````

````
sudo su - jenkins
````

----------------- Install or update the AWS CLI --------------
````
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
./aws/install -i /usr/local/aws-cli -b /usr/local/bin
aws --version
````

------------------- Install eksctl on Ubuntu Linux ---------------
````
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
````

---------------- Installing or updating kubectl -------------------

````
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl.sha256
sha256sum -c kubectl.sha256
openssl sha1 -sha256 kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
kubectl version --short --client
````
------------------------ AWS Configure --------------------------

create IAM user with administration access
````
aws configure
````
AWS Access Key ID [None]: AKIATJQNTRM7NOULDJU6
AWS Secret Access Key [None]: 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
Default region name [None]: ap-south-1
Default output format [None]: json

------------------------------ Cluster creation ----------------------- 

````
eksctl create cluster --name naresh-test-cluster --region ap-south-1 --nodegroup-name my-nodes --node-type t3.small --managed --nodes 2
````
````
eksctl get cluster --name naresh-test-cluster --region ap-south-1
````
````
aws eks update-kubeconfig --name naresh-test-cluster --region ap-south-1
````

Added new context arn:aws:eks:ap-south-1:226588068670:cluster/demo-eks to (/root/.kube/config)--> copy the path
(crete credencials with this config file)
````
cat /root/.kube/config 
````
````
kubectl get nodes
````
````
kubectl get deployments
````
````
kubectl get service
````
````
eksctl delete cluster --name naresh-test-cluster --region ap-south-1
````

## ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD

1 ) First, create a namespace
    
    ````
    kubectl create namespace argocd
    ````
2 ) Next, let's apply the yaml configuration files for ArgoCd
    
    ````
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ````
3 ) Now we can view the pods created in the ArgoCD namespace.
    
    ````
    kubectl get pods -n argocd
    ````
4 ) To interact with the API Server we need to deploy the CLI:
    ````
    curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
    rm argocd-linux-amd64
    ````
5 ) Expose argocd-server
    $ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

6 ) Wait about 2 minutes for the LoadBalancer creation
    ````
    kubectl get svc -n argocd
    ````
7 ) Get pasword and decode it.
    ````
    kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
    ````
    ````
    echo WXVpLUg2LWxoWjRkSHFmSA== | base64 --decode
    ````
## Add EKS Cluster to ArgoCD

8 ) login to ArgoCD from CLI
    ````
    argocd login a2255bb2bb33f438d9addf8840d294c5-785887595.ap-south-1.elb.amazonaws.com --username admin
    ````
    ````
    argocd cluster list
    ````
9 ) Below command will show the EKS cluster
     ````
     kubectl config get-contexts
     ````
10 ) Add above EKS cluster to ArgoCD with below command
     ````
     argocd cluster add arn:aws:eks:ap-south-1:074946469941:cluster/naresh-test-cluster --name aresh-test-cluster
     ````
    ````
    kubectl get svc
    ````
    ````
    argocd cluster list
    ````

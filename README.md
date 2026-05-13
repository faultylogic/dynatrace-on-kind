# dynatrace-on-kind
repo containing the bare min to get Kind-k8 running with ingress and Dynatrace 


#Install Docker
sudo apt-get update
sudo apt install -y docker.io
sudo usermod -aG docker $USER && newgrp docker

# install brew package manager
sudo apt install build-essential procps curl file git -y
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"


#required dependencies 
brew install go
brew install kind
brew install helm
brew install cloud-provider-kind
brew install kubectl

#create mount point for PV's for the cluster
sudo mkdir /mnt/root


kind create cluster --name kindk8 --config kind-cluster.yaml
#set up ngix for ingress 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
# race condition for tthe dedployment of the pods before applying nginx
kubectl apply -f nginx.yaml


helm install dynatrace-operator oci://public.ecr.aws/dynatrace/dynatrace-operator \
--create-namespace \
--namespace dynatrace \
--atomic

kubectl apply -f dynakube.yaml

#would recommend adding an alias for the lazy 
#alias k="kubectl"

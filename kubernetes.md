#Git
git clone https://github.com/yukkie1973
git add kubernetes.md
git config --global user.email "yukkie1973@gmail.com"
git config --global user.name "yukkie1973"
git commit -m "chap02"
git push

#Chapter 02
#ACR
#(1)Create Resistry
ACR_NAME=sampleACRRegistry
ACR_RES_GROUP=$ACR_NAME
az group create --resource-group $ACR_RES_GROUP --location japaneast
az acr create --resource-group $ACR_RES_GROUP --name $ACR_NAME -sku Standard --location japaneast

#(2)Download Sample
git clone https://github.com/ToruMakabe/Understanding-K8s
cd Understanding-K8s/chap02/

#(3)Build Container Image
az acr build --registry $ACR_NAME --image photo-view:v1.0 v1.0/
az acr build --registry $ACR_NAME --image photo-view:v2.0 v2.0/

#(4)Confirm Container Image
az acr repository show-tags -n $ACR_NAME --repository photo-view

#AKS
#(1)ACR+AKS
ACR_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
$SP_NAME=sample-acr-service-principal
SP_PASSWD=$(az ad sp create-for-rbac --name $SP_NAME --role Reader --scopes $ACR_ID --query password --output tsv)
APP_ID=$(az ad sp show --id http://$SP_NAME --query appId --output tsv)
echo $APP_ID
echo $SP_PASSWD

#(2)Create AKS Cluster
AKS_CLUSTER_NAME=AKSCluster
AKS_RES_GROUP=$AKS_CLUSTER_NAME
az group create --resource-group $AKS_RES_GROUP --location japaneast
az aks create --name $AKS_CLUSTER_NAME --resource-group $AKS_RES_GROUP --node-cout=3 --kubernetes-version 1.11.4 --node-vm-size Standard_DS1_v2 --generate-ssh-keys --service-princepal $APP_ID --client-secret $SP_PASSWD

#(3)Set AKS Credential
az aks get-credentials --admin --resource-group $AKS_RES_GROUP --name $AKS_CLUSTER_NAME

#(4)kubectl
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

kubectl get node -o=wide
kubectl get pod -o=wide
kubectl get deploy -o=wide
kubectl get hpa -o=wide
kubectl cluster-info
kubectl describe pod xxxx

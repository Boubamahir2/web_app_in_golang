curl https://storage.googleapis.com/kubernetes-release/release/stable.txt > ./stable.txt
export KUBECTL_VERSION=$(cat stable.txt)
curl -LO https://storage.googleapis.com/kubernetes-release/release/$KUBECTL_VERSION/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
mkdir -p ~/.kube
ln -sf "/mnt/c/users/$USER/.kube/config" ~/.kube/config
rm ./stable.txt

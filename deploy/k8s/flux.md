flux create source git flux-system  --git-implementation=libgit2  --url=ssh://git@ssh.dev.azure.com/v3/tjip/AbnAmro/Experimental --branch=dev-cluster --ssh-key-algorithm=rsa --ssh-rsa-bits=4096 --interval=1m


flux create kustomization flux-system --source=flux-system --path="./deploy/k8s" --prune=true --interval=10m


flux export source git flux-system > ./deploy/k8s/flux/gotk-sync.yaml

flux export kustomization flux-system >> ./deploy/k8s/flux/gotk-sync.yaml

cd ./deploy/k8s/flux/ && kustomize create --autodetect
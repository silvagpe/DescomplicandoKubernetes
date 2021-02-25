# Links Importantes

Tipos de topologias de K8s multi-master: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/

Instalação kubeadm, kubelet e kubectl: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

Instalação Kubernetes multi-master: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

HAproxy: https://www.haproxy.org/



## Como dar permissão de sudo

Subir para o nível de su

```
su -
```

Instalar o sudo 
```
apt-get install sudo -y
```

Dar permissão para o grupo
```
usermod -aG sudo silva
```

Conferir o arquivo /etc/sudoers
```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
```

## Autocomplete
```
source /etc/bash_completion
source <(kubectl completion bash)

echo "source <(kubectl completion bash)" >> ~/.bashrc
```
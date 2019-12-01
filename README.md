## selfhosted-services
The services that will be running on hubbe.club
In kubernetes, once I'm done porting and testing

### k8s Cluster Setup
- Set up control-plane single-node "cluster" with `kubeadm-init.sh` script
    - Note that this script is likely Arch Linux specific
    - This also sets up `Calico` for pod networking
    - Default pod CIDR is 10.0.0.0/16, adjust as needed
    - This also sets up docker to use systemd as its cgroup driver
    - This also enables kernel source address verification

### Storage setup
- NFS shares for things in `volumes` dir need to be created
    - Sadly, it seems one PV/PVC pair can't be mounted into one pod multiple times
        - Ideally, I'd want one PV/PVC pair that just says 192.168.1.213:/mnt/hubbe/array
        - And then mount that multiple times in one pod with different subPaths
            - Tried that, k8s hung when creating, waiting for things to mount
    - Must be accessible from the IPs nodes in the k8s cluster
    - Somewhat unsure how to do permissions properly? This works, though:
        - Should use `all_squash` on NFS server to squash all access to a known UID/GID pair
        - Containers may also need to runAsUser with this UID to not break things when trying to `chown`

### Ingress setup
- Set up `cert-manager` for automated cert fetching/renewing
    - Check `cert-issuer/*.yaml.example` files
    - Run `./setup-cert-manager.sh`
- Deploy nginx ingress controller with `./setup-nginx.sh`
    - Current configuration assumes a single wildcard cert, `nginx-ingress/tls` for all sites
    - Issued by LetsEncrypt, solved by CloudFlare DNS verification
    - See `nginx/certificate.yaml` for certificate request fulfilled by `cert-manager`

### Apps setup
- Some apps need iGPU acceleration (jellyfin)
    - See https://github.com/intel/intel-device-plugins-for-kubernetes/tree/master/cmd/gpu_plugin
- Deploy volumes (PV/PVC) with `kubectl apply -f volumes/`
- Create valid secrets from `*-secret.yaml.example` files in `apps` dir
    - Save as `apps/*-secret.yaml`
- Create MariaDB initial database/user creation SQL from `apps/mariadb-init-config.yaml.example`
    - Save as `apps/mariadb-init-config.yaml`
- Deploy apps
    - All apps can be deployed simply with `kubectl apply -f apps/`
    - If deploying single apps, remember to also deploy related secrets

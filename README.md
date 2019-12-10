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
    - Must be accessible from the IPs of the nodes in the k8s cluster
    - NFS export for `service-data` should use `no_root_squash`, as some apps run as root and may fail if NFS doesn't give files the expected owner
    - NFS is a bit slow, though. Might be better to do some ZFS zvol + iscsi thing.

### Ingress setup
- Set up `cert-manager` for automated cert fetching/renewing
    - Check `cert-issuer/*.yaml.example` files
    - Run `./setup-cert-manager.sh`
- Deploy nginx ingress controller with `kubectl apply -f nginx/`
    - Current configuration assumes a single wildcard cert, `ingress-nginx/tls` for all sites
    - Issued by LetsEncrypt, solved by CloudFlare DNS verification
    - See `nginx/certificate.yaml` for certificate request fulfilled by `cert-manager`

### Apps setup
- Some apps need iGPU acceleration (jellyfin)
    - See https://github.com/intel/intel-device-plugins-for-kubernetes/tree/master/cmd/gpu_plugin
- Check NFS server IPs and share paths in `volumes` directory
    - Deploy volumes (PV/PVC) with `kubectl apply -f volumes/`
- Create config for `hubbot` from `hubbot-config.yaml.example`
- Create config for `roundcubemail` from `roundcubemail-config.yaml.example`
- Optional:
    - Set new MYSQL_PASSWORD environment variables for `nextcloud` and `freshrss`
- Set PUID, PGID, and TZ variables in `apps/0-linuxserver-envs.yaml`
- Deploy apps
    - All apps can be deployed simply with `kubectl apply -f apps/`
    - If deploying single apps, remember to also deploy related configs
        - Most things need the `apps/0-linuxserver-envs.yaml` ConfigMap

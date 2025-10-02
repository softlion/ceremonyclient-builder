# Quilibrium helm chart

## Install

Create a PVC for persistant storage, called `yourPVCName` below.  
Then install the release:
```shell
helm upgrade quilibrium helm/quilibrium --install \
  --set persistence.existingClaim=yourPVCName
```

And check the logs for health  
```shell
kubectl logs -f quilibrium-xxxxxxxxxx-xxxxx
```

## Commands

See balance/earnings  
Note: you must wait 5mn after the node has (re)started to see anything.  
```shell
kubectl exec quilibrium-xxxxxxxxxx-xxxxx -- node -node-info
```

Sample result:  
```text
Signature check passed
Peer ID: -------------------------------------------
Version: 2.0.3-p4
Max Frame: 55586
Prover Ring: 5
Seniority: 2737593
Owned balance: 0.9536040931000 QUIL
Note: bridged balance is not reflected here, you must bridge back to QUIL to use QUIL on mainnet.
```

## Host setup (Debian)

```bash
sudo nano /etc/sysctl.d/99-quilibrium.conf

net.core.rmem_max=7500000
net.core.wmem_max=7500000

sudo sysctl --system
```

## Docker Build
Built by github action, it uses the official signed binaries and a slim debian container.  
It is published on [docker hub](https://hub.docker.com/r/vapolia/quilibrium/tags)

## Advanced configuration

The root of the PV contains:
- A `config.yml` file, in which you should set `maxFrames` to `1001` instead of the default `-1`. 
That will prevent the node from taking 2TB of disk space after a few days.
- A 'key.yml' file, that you should backup as soon as it is created.
- A `store` folder, which can be deleted at any time to free up disk space (stop and restart the node before and after deleting files there).

### Cleaning the store folder

Stop the node temporarily, clean the store folder, then restart the node:  

```shell
kubectl scale deployment quilibrium --replicas=0
sudo find /your-specific-path/store -mindepth 1 -delete
kubectl scale deployment quilibrium --replicas=1
```

### Recovering from failure

```shell
kubectl delete pods --field-selector=status.phase!=Running -A
```

## Links

Sources:  
https://github.com/QuilibriumNetwork/ceremonyclient

Install Guides and tools:  
https://github.com/0xOzgur/QuilibriumTools
https://github.com/demipoet/quilibrium-guide-github

Videos:  
https://www.youtube.com/watch?v=GfEqWw-p_yU

Dashboard:  
https://github.com/fpatron/Quilibrium-Dashboard

Advanced configuration:  
https://docs.quilibrium.com/docs/run-node/advanced-configuration/

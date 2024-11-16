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

## Links

Sources:  
https://github.com/QuilibriumNetwork/ceremonyclient
https://source.quilibrium.com/quilibrium/ceremonyclient

Install Guides and tools:  
https://github.com/0xOzgur/QuilibriumTools
https://github.com/demipoet/quilibrium-guide-github

Videos:  
https://www.youtube.com/watch?v=GfEqWw-p_yU

Dashboard:  
https://github.com/fpatron/Quilibrium-Dashboard

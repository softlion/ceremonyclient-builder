# Quilibrium helm chart

## Install

Create a PVC for persistant storage, called `yourPVCName` below.  
Then install the release:
```shell
helm upgrade quilibrium helm/quilibrium --install \
  --set persistence.existingClaim=yourPVCName
```

And check the logs for errors  
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
Version: 1.4.19
Max Frame: 61
Peer Score: 0
Note: Balance is strictly rewards earned with 1.4.19+, check https://www.quilibrium.com/rewards for more info about previous rewards.
Unclaimed balance: 0.029400000000 QUIL
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

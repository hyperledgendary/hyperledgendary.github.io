---
layout: default
title: Developing with Hyperledger Fabric External Chaincode
parent: Tutorials
---

# Aim:

To demonstrate how to setup a development environment to use external chaincode

# Objectives

- Clarify the difference between external launchers and external-as-a-server chaincode
- How to package a chaincode in a docker container
- How to use that with microfab
- How to connect that to the IBP vscode extension.

## Clarification

The establish process of deploying chaincode is to package this (in Fabric 2 as a tgz file) and install it on the peer. Then after some approving and commiting this, the peer will spin up a docker container itself to run the chaincode within. Choosing the type of docker container based on the language you've said the implemention is in.

Fabric 2.2 introduced the concept of the External Builders and Launchers that let you control exactly how the this process of the spinning up the chaincode worked.

The last part of the this process is to set the [chaincode running](https://hyperledger-fabric.readthedocs.io/en/release-2.2/cc_launcher.html#bin-run). When the chaincode starts, the libraries call back to the peer to start communications.

This subtle, but important impact of this is the peer is still very involved in when the chaincode starts, and is tightly coupled with the deployment of the peers themselves. By this I mean the run script has to be able to find and start the chaincode. 

This is solved by using the 'Chaincode-as-a-server' feature (often though called External Chaincode Service... the terminology gets a bit 'loosey-goosey' here which is a problem).

With this, the chaincode is run as a 'server' in it's own right. Typically a Docker container will be run, which will listen on a port for the pPeer to 'say hello'.

The peer knows where to call to 'say hello' as you provide a 'proxy chaincode' that contains the IP/Port and TLS configuration. 

This is the approach described here.

> TL;DR;  External Chaincode Server, lets you have freedom to start a chaincode how and when you like, just telling the peer where it is and making sure it was started before first transaction.


In this demo scenario, we'll be running two docker containers. As these need to interact we need to create a dedicated docker network

```bash
docker network create audit_network
```

Startup the Fabric Infrastructure, we're using MicroFab here as it's a single container and fast to start. Plus it already has the configuration required within it to start external chaincodes.

```bash
export MICROFAB_CONFIG='{
    "endorsing_organizations":[
        {
            "name": "Ampretia"
        }
    ],
    "channels":[
        {
            "name": "auditnet",
            "endorsing_organizations":[
                "Ampretia"
            ]
        }
    ],
    "capability_level":"V2_0"
}'

docker run --name microfab --network audit_network --rm -ti -p 8080:8080 -e MICROFAB_CONFIG="${MICROFAB_CONFIG}"  ibmcom/ibp-microfab

```

'Proxy' Chaincode

```json
{
  "address": "javacc.example.com:9999",
  "dial_timeout": "10s",
  "tls_required": false
}
```


npm install -g @hyperledgendary/weftility
curl -s http://console.127-0-0-1.nip.io:8080/ak/api/v1/components | weft microfab -w ./_cfg/_wallets -p ./_cfg/_gateways -m ./_cfg/_msp -f



```bash
curl -s https://raw.githubusercontent.com/hyperledgendary/fabric-builders/master/tools/pkgcc.sh > ./pkgcc.sh && chmod u+x ./pkgcc.sh
./pkgcc.sh -l javacc -t external connection.json
```

docker build -t java-cc-server .  



CHAINCODE_SERVER_ADDRESS=javacc.example.com:9999
CHAINCODE_ID=javacc:30e755c332d780fb6d5c55e9c48e5865fe22df2e642ecc228ef37c0d523793f9


docker run -it --rm --name javacc.example.com --hostname javacc.example.com --env-file chaincode.env --network=audit_network java-cc-server

ks

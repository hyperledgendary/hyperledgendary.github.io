---
layout: default
title: Caliper network_config.json file guide
parent: Performance
---

This is the <i>network_config.json</I> template used in the <a>https://hyperledger.github.io/caliper/v0.3.2/fabric-tutorial/tutorials-fabric-existing/</a> tutorial.
Follow the tutorial but instead of populating the <i>network_config.json</I> file, like specified in the tutorial, just insert your data where specified in the json code below.
<br/><br/>

```
{
    "version": "<version from the connection profile file>",
    "name": "<name from the connection profile file>",
    "caliper": {
        "blockchain": "fabric"
    },
    "clients": {
        "<identity name>": {
            "client": {
                "credentialStore": {
                    "path": "tmp/hfc-kvs/org1",
                    "cryptoStore": {
                        "path": "tmp/hfc-kvs/org1"
                    }
                },
                "clientPrivateKey": {
                    "path": "<path of the client private key>"
                },
                "clientSignedCert": {
                    "path": "<path of the client signed certificate>"
                },
                "connection": {
                    "timeout": {
                        "orderer": "<timeout value for the given node from the connection profile file>",
                        "peer": {
                            "endorser": "<timeout value for the given node from the connection profile file>"
                        }
                    }
                },
                "organization": "<organization from the connection profile file>"
            }
        }
    },
    "channels": {
        "<channel name>": {
            "created": true,
            "chaincodes": [
                {
                    "id": "<chaincode ID (label from peer lifecycle chaincode queryinstalled command)>",
                    "version": "<chaincode version>"
                }
            ]
        }
    },
    "organizations": {
        "<organization name from the connection profile file>": {
            "mspid": "<mspid from the connection profile file>",
            "peers": [
                "<peers from the connection profile file>"
            ],
"<optional certificateAuthorities>": [
    "<certificateAuthorities from the connection profile file>"
]
        }
    },
    "peers": {
        "<peer name from the connection profile file>": {
            "grpcOptions": {
                “<grpcOptions from the connection profile file>”
            },
            "url": "<url from the connection profile file>"
        }
    },
    "<optional certificateAuthorities>": {
        <certificateAuthorities from the connection profile file>
    }
}
```


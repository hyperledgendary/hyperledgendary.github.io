---
layout: default
title: Hyper-speed Smart Contract Developing
parent: Tutorials
---

CURRENTLY IN DRAFT

## Code-Debug-Test your contracts at Hyper Speed!

Aim is to show you how to code, debug, and then test your contracts in the fastest possible time. The only way to do it faster is write the code 100% correctly first time :-) 


> ***For this Hyper-speed tutorial, we're using the very latest microfab and 2.4.0-beta of Hyperledger Fabric. The downside of this is that you'll need to rebuild the microfab docker image.***
>
>1. Clone the `fabric-2.` branch of the repo. `git clone -b fabric-2.4 https://github.com/IBM-Blockchain/microfab.git`
>2. Run `docker build -t ibmcom/ibp-microfab ./microfab` to build a local copy


### How this works?

We're using `microfab` which is a single docker container that can spin up a set of Peers/Orderers/CAs in seconds. It uses docker but doesn't have the load that the multi container approaches have. Whilst this is not great for productions, for development it's perfect.

It's helpful to have at least 3 command shell/windows open. One for running the chaincode in, one for running 'peer' commands in, and in one start microfab, like this.

```bash
export MICROFAB_CONFIG='{
    "endorsing_organizations":[
        {
            "name": "DigiBank"
        }
    ],
    "channels":[
        {
            "name": "assetnet",
            "endorsing_organizations":[
                "DigiBank"
            ]
        }
    ],
    "capability_level":"V2_0"
}'

docker run --name microfab --rm -ti -p 8080:8080 -e MICROFAB_CONFIG="${MICROFAB_CONFIG}"  ibmcom/ibp-microfab

```

This should start in a few seconds; you can add more organizations, channels etc. You can also enable TLS!  (though that disables the chaincode dev mode feature of the peer we'll be using here.)

Re-introduced in Fabric v2.3, the 'chaincode-dev-mode' allows chaincodes to be started manually.  This does have strong similarities to the chaincode-as-a-server feature.

### Get the client side configuration

To talk to the Fabric Peers etc, to install chaincode and run client applications some configuration needs to be extracted from microfab container.

This can be done with a simple curl command `curl -s http://console.127-0-0-1.nip.io:8080/ak/api/v1/components` Though you would need to parse out the JSON yourself.

Luckily, there's a helper tool that does for you, and also creates the correct MSP folder structure for the peer commands, and wallet/connection information for client applications

```bash
# note best used with Node v12
npm install -g @hyperledgendary/weftility
```

When this has installed, you can re-issue the `curl` command and pipe the output into the `weft` tool. In this example bellow, the `_cfg` directory is being used to hold all the configuration information, and you can see the output showing what has been created.

```bash
curl -s http://console.127-0-0-1.nip.io:8080/ak/api/v1/components | weft microfab -w ./_cfg/microfab/_wallets -p ./_cfg/microfab/_gateways -m ./_cfg/microfab/_msp -f
Gateway profile written to : /home/matthew/github.com/hyperledgendary/contract-as-a-server/infrastructure/dev-microfab/_cfg/microfab/_gateways/digibankgateway.json
Added identity under label digibankadmin to the wallet at /home/matthew/github.com/hyperledgendary/contract-as-a-server/infrastructure/dev-microfab/_cfg/microfab/_wallets/DigiBank
Added identity under label digibankcaadmin to the wallet at /home/matthew/github.com/hyperledgendary/contract-as-a-server/infrastructure/dev-microfab/_cfg/microfab/_wallets/DigiBank
Added identity under label ordereradmin to the wallet at /home/matthew/github.com/hyperledgendary/contract-as-a-server/infrastructure/dev-microfab/_cfg/microfab/_wallets/Orderer

Environment variables:

For digibankadmin @  DigiBank use these:

export CORE_PEER_LOCALMSPID=DigiBankMSP
export CORE_PEER_MSPCONFIGPATH=/home/matthew/github.com/hyperledgendary/contract-as-a-server/infrastructure/dev-microfab/_cfg/microfab/_msp/DigiBank/digibankadmin/msp
export CORE_PEER_ADDRESS=digibankpeer-api.127-0-0-1.nip.io:8080

For digibankcaadmin @  DigiBank use these:

export CORE_PEER_LOCALMSPID=DigiBankMSP
export CORE_PEER_MSPCONFIGPATH=/home/matthew/github.com/hyperledgendary/contract-as-a-server/infrastructure/dev-microfab/_cfg/microfab/_msp/DigiBank/digibankcaadmin/msp
export CORE_PEER_ADDRESS=digibankpeer-api.127-0-0-1.nip.io:8080
```

We want to act as the `digibankadmin` here so copy the first set of `export....` statments in your current shell.

```bash
export CORE_PEER_LOCALMSPID=DigiBankMSP
export CORE_PEER_MSPCONFIGPATH=/home/matthew/github.com/hyperledgendary/contract-as-a-server/infrastructure/dev-microfab/_cfg/microfab/_msp/DigiBank/digibankadmin/msp
export CORE_PEER_ADDRESS=digibankpeer-api.127-0-0-1.nip.io:8080
```

You're now able to run the `peer` commands to manage chaincode etc. 


### Time to start the chaincode

Normally the peer would build and start the chaincode container for you. Here, you do this yourself. Three key points

- Remember to build your chaincode first
- Set the `--peer.address` and `chaincode-id-name` properties
- Remember the `chaincode-id-name` you used!

Start up the chaincode.

For a node.js contract this would be as follows (do this in the 'chaincode' shell)

```bash
npm start -- --peer.address=digibankpeer-chaincode.127-0-0-1.nip.io:8080 --chaincode-id-name=contract:1
#  some output removed here...
2021-10-19T08:36:55.179Z info [c-api:lib/chaincode.js]                            Registering with peer digibankpeer-chaincode.127-0-0-1.nip.io:8080 as chaincode "contract:1"
2021-10-19T08:36:55.471Z info [c-api:lib/handler.js]                              Successfully registered with peer node. State transferred to "established"
2021-10-19T08:36:55.472Z info [c-api:lib/handler.js]                              Successfully established communication with peer node. State transferred to "ready"
```
That's the chaincode started.. at any point you can stop this running, update it and restart. Or attach a debugger. 

The only caveat is make sure you do the next two steps first.

### Chaincode Approve and Commit

The Approve and Commit phases of the chaincode lifecycle are still required (note not the install). From the 'peer' command window

```bash
peer lifecycle chaincode approveformyorg -o orderer-api.127-0-0-1.nip.io:8080 --channelID assetnet --name contract --version 1 --sequence 1 --waitForEvent --package-id contract:1
peer lifecycle chaincode commit -o orderer-api.127-0-0-1.nip.io:8080 --channelID assetnet --name contract --version 1 --sequence 1
peer chaincode query -o orderer-api.127-0-0-1.nip.io:8080 --channelID assetnet --name contract -c '{"function":"org.hyperledger.fabric:GetMetadata","Args":[]}' | jq
```

The last `peer chaincode query` command is useful to check that everything is up and working.

Pay attention to the `package-id` that MUST match the `chaincode-id-name` the chaincode was started with. Also note the the name of the chaincode. In this case `contract` it must be the same in all the `peer` commands


Once this has been done, you can start stop, debug as you wish the chaincode running in the other window. You don't need to to re-issue these approve/commit unless you need to adjust the endorsement policies. Or you shut everything down and want to start again.


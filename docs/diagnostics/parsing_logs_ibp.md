---
layout: default
title: Parsing Chaincode Logs from IBM Blockchain Platform
parent: Good Practice
---
# IBP Chaincode Log Parse

*tl;dr; simple command line tool to parse the chaincode logs from IBP to more readable form*

```
npm install -g @ampretia/ibp-chaincode-log
kubectl --namespace=nbe307c logs org1peer-7ccbd7c5f6-7k55d -c chaincode-logs > ibp.log
ibpccl --file ibp.log 
```

Pre-requsites: Node12, and a IBP chaincode_logs log file. Mostly easily obtained via `kubectl` command line.

(this is document is copied from [the tool's git repo](https://github.com/ampretia/ibp-chaincode-log))

## When to use?

When running the test networks on your local machine, the output of the chaincode (smart contract) containers comes to the stdout/stderr of the docker container. You can watch this with `docker logs -f xxxx` as you want.

IBP is deployed into a K8S cluster, and there are few more steps to get to the logs; they are also wrapped in a log file themselves. 

## Getting a IBP log file

1. Best to setup the `kubectl` command line, from the IBM Kubernetes Cluster control panel, there's a 'Access via CLI' option that shows information on how setup the command line. 
2. Once configured, you will need to know the namespace in Kuberneters where the pods are that are running the peers for your IBP instance. 
   - either navigate to the IBP Console, and click on a peer, and look under the 'Info' tab. This will give a number of URLs, the first part of these URLs will be the namespace.
   - or run the `kubectl get namespaces` command and guess. Will be something like `nbe307c`
3. As an example let's assume that the name is `nbe307c`. List the pods that are part of this namespace.

```
kubectl --namespace=nbe307c get pods
NAME                                    READY   STATUS    RESTARTS   AGE
ibp-operator-58595bff67-bqlk7           1/1     Running   0          13d
orderingorgca-576b46cfc-kw4dj           1/1     Running   0          10d
orderingservicenode1-5f4cd9566f-4bbjj   2/2     Running   0          10d
org1ca-9b74dd4fd-7r5zh                  1/1     Running   0          10d
org1peer-7ccbd7c5f6-7k55d               5/5     Running   0          10d
org2ca-6dcbf66d57-nnlmf                 1/1     Running   0          10d
org2peer-65846466c4-8fxlg               5/5     Running   0          10d
```

This is the standard 2 organization network - and I know that I've install my contract onto both peers. Choose one of the pods that represents the peer, eg `org1peer-7ccbd7c5f6-7k55d`

4. You can access the logs for the chaincode containers on this peer. All the logs are accessed via one container.  Best to pipe the output to a file.

```
kubectl --namespace=nbe307c logs org1peer-7ccbd7c5f6-7k55d -c chaincode-logs > ibp.log

ls -l ibp.log
-rw-rw-r-- 1 matthew matthew 145652 Jul  3 13:22 ibp.log
```

5. This file is plain text, and is readable via text editor. The logs though have been wrapped in JSON so it's a lot easier to extact them, hence this tool.

Install it with  `npm install -g @ampretia/ibp-chaincode-log`

6. Run this with the `--file` option and the output will be parsed.

```
ibpccl --file ibp.log
```

The output will be more familiar now, and easier to read.

## Tips

The main issue with contracts not deploying correctly is some problem with they way they are built, maybe a compilation error, or missing file etc.

If you look in the parsed log, the container name is on each line after the timestamp, here it is 'nostalgic-dijkstra', and you can see the welcome message from Gradle. 

(this naming convetioned is the the default of Docker containers if no name is given)

```
[2020-07-02T13:54:21.000Z nostalgic_dijkstra  ] Welcome to Gradle 4.10.2!
```

Any container named like this will be one of the containers used to build the contract, and is a good starting point for errors.


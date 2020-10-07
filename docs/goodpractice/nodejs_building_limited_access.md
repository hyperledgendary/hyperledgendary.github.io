---
layout: default
title: Node.js Contracts Building with limited internet access
parent: Good Practice
---

Installing Node.js applications and libraries is inherently reliant on being able to access code repositories on the internet. Many other languages also follow this to a greater or lesser extent.  But there are many and quite legitimate cases where the access to the internet as a whole is controlled via firewalls proxys etc.

If you have a Fabric installation in such a circumstance then it is important to consider if Node.js is the best way to proceed.
This document is advice to help with this situation. If the options below are not applicable to your situation for whatever reason then it is best to consider a different language. It is not possible to modify Fabric, as the underlying cause is partly related to how node.js works

> **Recommendation #0 : Consider the full lifecycle of your contract from dev to production. Is the language choice suitable?**


## Starting Facts

Important to clear up that there is a subtle but very key difference between the Hyperledger Fabric v1.4 and v2.0 Node.js Contract libraries. v1.4 used two native (C) libraries, whereas v2.0 is 100% JavaScript.  Installing both JavaScript and Native libraries (ideally) needs internet access, but it is harder to work around the native use case (but not impossible).

> **Recommendation #1 :  Use v2 Node.js Contract Libraries if possible**

### When can you use the v2 Node.js libraries?

Check the document [COMPATIBILITY.md](https://github.com/hyperledger/fabric-chaincode-node/blob/master/COMPATIBILITY.md) for full information. This document describes the Hyperledger Fabric open source compatibility.  

The key element is that the environment created for running the chaincode is *Node 12*. In terms of the network communication to the peer, the version of peer is not relavent. Where the version of the peer *is* relavent is the impact it has on how runtime for the chaincode is created. In a pure open source Fabric network, this could be configured as described in the above document. For a SaaS offering, such as IBP, this is likely not to be possible. *By default* a v1.4 Peer will create a Node v8 environment, therefore only v1.4 libraries can be used. Similarly, *by default* a v2 Peer will create a Node v12 environment that can run v2.0 (and v1.4 node libraries).

## Handling the JavaScript case

### Locking the versions
There is a lot of literature online about how to lock the dependencies of the node modules; there is also a lot of different opinions not least due in part to changes on how npm works. This won't be debated here, but the conclusion is that you MUST include the `shrinkwrap` file in your production contract deployment. At the least include the `package-lock.json`.  If not, then whenever the contract container is started, and `npm install --production` is run for you a different set of packages COULD be installed. 

`npm shrinkwrap` does have it's flaws (includes all the dev-dependencies) but it is the only way to get an accurate installation. 

> **Recommendation #2 : After testing in QA, shrinkwrap your node.js contracts**

### Local npmjs repository

Have a local or corporate npmjs repository is a way of installing node.js without internet access.  

To use a different repository add a `.npmrc` file to the root directory of the contract (so it's a peer of the `package.json`).  This needs to contain the URL of the local repository. We use [Verdaccio](https://verdaccio.org/) during development, and also during integration test. We can publish the next version of the module locally and then check a realistic scenario.

As an example
```
registry=http://localhost:4873
```

Depending on the local npmjs server used, you should check the exact URL and configuration needed.. eg TLS. Check the [npm](https://docs.npmjs.com/configuring-npm/npmrc.html) documentation on `.npmrc`

The local npmjs server will need seeding however with the modules it needs to cache. Whilst it's possible to look and for the maintainers of the Fabric Chaincode to know what the Fabric node modules use directly, we can't produce a complete list that is accurate for all the dependency of dependencies - this will change over time as fixes are released, security patches etc. In addition, there will probably be node modules that the contract itself uses, and we need to let npm sort out any dependencies that might overlap.

The best approach is therefore to seed a repository by, using your *production shrinkwraped contract*, install using the registry set to the local server, but with it internet-connected. That will then cache all the node modules required for the real installation. That can then be disconnected and still able to serve and install. 

> **Recommendation #3 : Seed local npmjs repository and use .npmrc to refer to it**

## Native modules

> **Recommendation #4 : Use v2 Fabric Contract libraries, to avoid this issue altogether**

If you are not able to use the v2 libraries, then it is possible to manage the native code. There are two native node modules in use for X509 parsing, and for the grpc connectivity. The source code is downloaded from npmjs for these, and then is rebuilt locally. Getting the source is covered by the local npmjs registry.

There are two additional cases however

- Node modules can be downloaded pre-built - but these are downloaded from a URL specified in each package - and this can be anywhere
    - but if this is not available, the installation falls back to rebuilding from source
- If rebuilding from source, the node.js headers will be needed for the node.js runtime in use, these are downloaded


For the x509 library, the source is small so is rebuilt every time and is quick.
For the grpc library, this is downloaded as a pre-built binary. If this is not accessible or found the code is rebuilt from source.  Note though that this is not quick.

> **Recommendation #5 : Monitor and increase the timeout of the deploy of the contract to allow for compilation time**

To work-around the headers download, for the version of node that will build the code, goto [htps://nodejs.org/dist/](htps://nodejs.org/dist/) and find the version you are using. Then find the file with 'headers' in it.  eg for Node 8.17.0 the full path is [https://nodejs.org/dist/v8.17.0/node-v8.17.0-headers.tar.gz](https://nodejs.org/dist/v8.17.0/node-v8.17.0-headers.tar.gz).

Check the build logs of a real npm install of the contract to check exactly the version of node.js that is being used. Depending on how Fabric has been configured this could be different from the default.

Take this `node-vx.x.x-headers.tar.gz` and put this in the root of your module, and it MUST be included in the final package if you publish it. It is not a large file so should not give any issues with size.

In your contract's `pacakge.json` add this line to reference this file (change as needed)

```
  "dist-url": "node-v8.17.0-headers.tar.gz"
```

> **Recommendation #6 : Check the version of node and download a local copy of the node.js headers**

When the module is then rebuilt the local copy of the headers will be used, there is no interent access.  It is likely there will be failed internet access in an attempt to download the prebuilt library, but from the installation perspective, everything will proceed.


## Summary

1. You can use node.js Contracts in a constrained internet environment.  But they were not specifically designed with this in mind.

2. Consider if the ways of working around this are applicable in your situation now and in the future, and factor this into your choice of programming language

3. Run a local npmjs registry seeded with the modules installed from a shrink-wrapped version of your production contract

4. Use v2.0 Fabric Contract libraries where ever possible.

5. If you need to use v1.4, the native code will need to be recompiled. Check your version of node.
js and download the headers. Monitor the installation time, as it will be longer and may required configuration to allow for this.

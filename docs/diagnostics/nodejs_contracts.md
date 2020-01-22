---
layout: default
title: Node.js Contract Logging
parent: Diagnostics
---

# Logging from the Contract code

## Output

Logging output goes to stdout; as this is a docker container by default, this is easily captured using standard docker tools.

The following levels are used.

- CRITICAL
- ERROR
- WARNING
- DEBUG
- INFO

## Enabling Logging

In the `docker-compose.yml` environment variables of the peer 

```
CORE_CHAINCODE_LOGGING_LEVEL=[CRITICAL|ERROR|WARNING|DEBUG|INFO]
```

In a transaction function in the contract

```javascript

    async setLogLevel(ctx,loglevel) {

        ctx.logging.setLevel(loglevel);

        const logger = ctx.logging.getLogger("a name of your choice");
        // or
        const logger = ctx.logging.getLogger(); // defaults to contract name


        logger.info('Updated the log level to ',loglevel);

    }

```
## Getting the Log level

In a transaction function in the contract

```javascript
    let loglevel = process.env.CORE_CHAINCODE_LOGGING_LEVEL;
```
## APIs that can be used in the contract

The 'logger' object that is returned is a [Winston Logger object](https://github.com/winstonjs/winston#logging).



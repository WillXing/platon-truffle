# Configuration

## Location

Your configuration file is called `truffle-config.js` and is located at the root of your project directory. This file is a Javascript file and can execute any code necessary to create your configuration. It must export an object representing your project configuration like the example below.

```javascript
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*" // 匹配任何网络
    }
  }
};
```

The default configuration ships with configuration for a single development network, running on `127.0.0.1:8545`. There are many other configuration options, detailed below.

```eval_rst
.. _Resolving naming conflicts on Windows:
```

## General options

```eval_rst
.. _build:
```

### build

Build configuration of your application, if your application requires tight integration with Truffle. Most users likely will not need to configure this option. See the [Build Processes](../advanced/build-processes.md) section for more details.

```eval_rst
.. _config_networks:
```

### networks

Specifies which networks are available for deployment during migrations, as well as specific transaction parameters when interacting with each network (such as gas price, from address, etc.). When compiling and running migrations on a specific network, contract artifacts will be saved and recorded for later use. When your contract abstractions detect that your PlatON client is connected to a specific network, they'll use the contract artifacts associated that network to simplify app deployment. Networks are identified through PlatON's `net_version` RPC call, as well as blockchain URIs.

The `networks` object, shown below, is keyed by a network name and contains a corresponding object that defines the parameters of the network. The `networks` option is required, as if you have no network configuration, Truffle will not be able to deploy your contracts. The default network configuration provided by `truffle init` gives you a development network that matches any network it connects to -- this is useful during development, but not suitable for production deployments. To configure Truffle to connect to other networks, simply add more named networks and specify the corresponding network id.

The network name is used for user interface purposes, such as when running your migrations on a specific network:

```bash
$ truffle migrate --network live
```

Example:

```javascript
networks: {
  development: {
    host: "127.0.0.1",
    port: 8545,
    network_id: "*", // match any network
    websockets: true
  },
  live: {
    host: "178.25.19.88", // Random IP for example purposes (do not use)
    port: 80,
    network_id: 1,        // PlatON public network
    // optional config values:
    // gas
    // gasPrice
    // from - default address to use for any transaction Truffle makes during migrations
    // provider - web3 provider instance Truffle should use to talk to the PlatON network.
    //          - function that returns a web3 provider instance (see below.)
    //          - if specified, host and port are ignored.
    // skipDryRun: - true if you don't want to test run the migration locally before the actual migration (default is false)
    // timeoutBlocks: - if a transaction is not mined, keep waiting for this number of blocks (default is 50)
      }
  }
}
```

For each network, if unspecified, transaction options will default to the following values:

* `gas`: Gas limit used for deploys. Default is `6721975`.
* `gasPrice`: Gas price used for deploys. Default is `100000000000` (100 Shannon).
* `from`: From address used during migrations. Defaults to the first available account provided by your PlatON client.
* `provider`: Default web3 provider using `host` and `port` options: `new Web3.providers.HttpProvider("http://<host>:<port>")` 
* `websockets`: You will need this enabled to use the `confirmations` listener or to hear Events using `.on` or `.once`. Default is `false`.

For each network, you can specify either `host` / `port` or `provider`, but not both. If you need an HTTP provider, we recommend using `host` and `port`, while if you need a custom provider such as `HDWalletProvider`, you must use `provider`.

#### Providers

The following network list consists of a local test network and an Infura-hosted Ropsten network, both provided by HDWalletProvider. Make sure you wrap `truffle-hdwallet` providers in a function closure as shown below to ensure that only one network is ever connected at a time.

```javascript
networks: {
  ropsten: {
    provider: function() {
       return new HDWalletProvider(mnemonic, "https://ropsten.infura.io/v3/YOUR-PROJECT-ID");
    },
    network_id: '3',
  },
  test: {
    provider: function() {
      return new HDWalletProvider(mnemonic, "http://127.0.0.1:8545/");
    },
    network_id: '*',
  },
}
```

If you specify `host` and `port` instead of `provider`, Truffle will create its own default HTTP provider using that host and port, and no minimal network connection will be opened, so there is no need to do the function wrapping workaround. That said, you wouldn't be able to use a custom provider in this case.

### contracts_directory

The default directory for uncompiled contracts is `./contracts` relative to the project root. If you wish to keep your contracts in a different directory you may specify a `contracts_directory` property.

Example:

To have Truffle find contracts in `./allMyStuff/someStuff/theContractFolder` (recursively) at compile time:

```javascript
module.exports = {
  contracts_directory: "./allMyStuff/someStuff/theContractFolder",
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*",
    }
  }
};
```


```note::
   In addition to specifying a relative path, you can also use globs/regular expressions to selectively compile contracts.
```

### contracts_build_directory

The default output directory for compiled contracts is `./build/contracts` relative to the project root. This can be changed with the `contracts_build_directory` key.

Example:

To place the built contract artifacts in `./output/contracts`:

```javascript
module.exports = {
  contracts_build_directory: "./output",
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*",
    }
  }
};
```

The built contract artifacts do not need to be inside the project root:

```javascript
module.exports = {
  contracts_build_directory: "../../../output",
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*",
    }
  }
};
```

Absolute paths will also work. This is not recommended though, as an absolute path may not exist when compiled on another system. If you use absolute paths on Windows, make sure to use double backslashes for paths (example: `/home/Users/Username/output`).

### migrations_directory

The default migrations directory is `./migrations` relative to the project root. This can be changed with the `migrations_directory` key.

Example：

```javascript
module.exports = {
  migrations_directory: "./allMyStuff/someStuff/theMigrationsFolder",
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*",
    }
  }
};
```

### mocha

Configuration options for the [MochaJS](http://mochajs.org/) testing framework. This configuration expects an object as detailed in Mocha's [documentation](https://github.com/mochajs/mocha/wiki/Using-mocha-programmatically#set-options).

Example:

```javascript
mocha: {
  useColors: true
}
```

## Compiler configuration

In the `compilers` object you can specify settings related to the compilers used by Truffle.

### solc

Solidity compiler settings. Supports optimizer settings for `solc`.
You may specify...

+ any solc-js version listed at [solc-bin](http://solc-bin.ethereum.org/bin/list.json). Specify the one you want and Truffle will get it for you.
+ a natively compiled solc binary (you'll need to install this yourself, links to help below).
+ a dockerized solc from one of images published [here](https://hub.docker.com/r/ethereum/solc/tags/).
+ a path to a locally available solc
+ a solc-js parser for faster docker and native compilations

Truffle config example:

```javascript
module.exports = {
  compilers: {
    solc: {
      version: <string>, // A version or constraint - Ex. "^0.5.0"
                         // Can also be set to "native" to use a native solc
      docker: <boolean>, // Use a version obtained through docker
      parser: "solcjs",  // Leverages solc-js purely for speedy parsing
      settings: {
        optimizer: {
          enabled: <boolean>,
          runs: <number>   // Optimize for how many times you intend to run the code
        },
        evmVersion: <string> // Default: "petersburg"
      }
    }
  }
}
```

For more information, please see the Solidity documentation on [Compiler Input and Output JSON Description](https://learnblockchain.cn/docs/solidity/using-the-compiler.html#json).

### external compilers

For more advanced use cases with artifact creation you can use the external compilers configuration. You can use this feature by adding a `compilers.external` object to your Truffle config:

```javascript
module.exports = {
  compilers: {
    external: {
      command: "./compile-contracts",
      targets: [{
         /* compilation output */
      }]
    }
  }
}
```

When you run truffle compile, Truffle will run the configured command and look for contract artifacts specified by targets.

This new configuration supports a couple of main use cases:

+ Your compilation command outputs Truffle JSON artifacts directly. If your compilation command generates artifacts directly, or generates output that contains all the information for an artifact, configure a target as follows:

```javascript
module.exports = {
  compilers: {
    external: {
      command: "./compile-contracts",
      targets: [{
        path: "./path/to/artifacts/*.json"
      }]
    }
  }
}
```

Truffle will execute your script, then expand the glob (*) and find all .json files in the listed path and copy those over as artifacts in the build/contracts/ directory.

+ Your compilation command outputs individual parts of an artifact, and you want Truffle to generate the artifacts for you. The above use case might not be sufficient for all use cases. You can configure your target to run an arbitrary post-processing command:

```javascript
module.exports = {
  compilers: {
    external: {
      command: "./compile-contracts",
      targets: [{
        path: "./path/to/preprocessed-artifacts/*.json",
        command: "./process-artifact"
      }]
    }
  }
}
```

This will run ./process-artifact for each matched .json file, piping the contents of that file as stdin. Your ./process-artifact command is then expected to output a complete Truffle artifact as stdout.

Want to provide the path as a filename instead? Add `stdin: false` to your target configuration.

+ You can also specify the individual properties of your contracts and have Truffle generate the artifacts itself.

```javascript
module.exports = {
  compilers: {
    external: {
      command: "./compile-contracts",
      targets: [{
        properties: {
          contractName: "MyContract",
          /* other literal properties */
        },
        fileProperties: {
          abi: "./output/contract.abi",
          bytecode: "./output/contract.bytecode",
          /* other properties encoded in output files */
        }
      }]
    }
  }
}
```

Specify `properties` and/or `fileProperties`, and Truffle will look for those values when building the artifacts.

To override the working directory for all specified paths and running commands, use the `workingDirectory` option. For instance, the following will run `./proj/compile-contracts` and read `./proj/output/contract.abi`:


```javascript
module.exports = {
  compilers: {
    external: {
      command: "./compile-contracts",
      workingDirectory: "./proj",
      targets: [{
        fileProperties: {
          abi: "./output/contract.abi",
          bytecode: "./output/contract.bytecode",
        }
      }]
    }
  }
}
```
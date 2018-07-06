# Setup BCH with Bitcore (bitcore-node v3.1.3)

## Requirments

- Bitpay's custom `bitcoin-abc` (https://github.com/bitprim/bitcoin-abc/tree/tag0.17.1-bitcore)
  - Bitcore-node uses special RPC commands ([list](https://github.com/BTCGPU/BTCGPU/commit/1c1d93de551d778c7e868ebe6cff66e53d09e712) of additional RPC commands) that `bitcoin-abc` needs to support.
  - `bitcoin-abc` needs to be built from source, the instructions on how to build it can be found under [`doc`](https://github.com/bitprim/bitcoin-abc/tree/tag0.17.1-bitcore/doc).

## Installation
Once you have the custom `bitcoind` from above, we can follow the standard `bitcore` installation guide:
```
npm install -g bitcore
bitcore create mynode-abc && cd mynode-abc
bitcore install osagga/insight-api#cash_v4 insight-ui
```
The script above will install the default bitcore (v3) from npm and install the customized `insight-api` (optional) and the standard `insight-ui`

## Configuration

The basic trick here is that we would run `bitcoin-abc` with `usecashaddr=0` and force it to use the legacy BTC addresses, and that way, the bitcore node will interact with `bitcoin-abc` as if it's a normal BTC node. 

(Optional) The another trick that I did is to modify the `insight-api` such that it converts the BTC legacy addresses back to BCH new addresses before serving api requests.

By default, if you use the standard `insight-api`, you can only use BTC legacy addresses. If you want to enable BCH address handling, consider using my fork on the api on `github.com/osagga/insight-api/tree/cash_v4`

### Config Samples
There are two main config file that we need to take care of:

- `bitcore-node.json`, this controls the `bitcore-node`
```json
{
    "network": "livenet",
    "port": 3001,
    "services": [
      "bitcoind",
      "insight-api",
      "insight-ui",
      "web"
    ],
    "servicesConfig": {
      "insight-ui": {
          "routePrefix": "",
          "apiPrefix": "api"
      },
      "insight-api": {
          "routePrefix": "api",
          "disableRateLimiter": true
      },
      "bitcoind": {
        "spawn": {
          "datadir": "./data", // Path to where bitcoind would sava data and load the config file.
          "exec": "FULL-PATH/TO/BITCOIND" // The one you built from source
        }
      }
    }
  }
```
- "data/bitcoin.conf" this controls the bitcoin-node (`bitcoin-abc`)
```conf
server=1

whitelist=127.0.0.1

# This can be anything you need
port=18449 

# This is IMPORTANT, without it, bitcoin-abc will use BCH addresses
usecashaddr=0 

txindex=1
addressindex=1
timestampindex=1
spentindex=1

zmqpubrawtx=tcp://127.0.0.1:28331
zmqpubhashblock=tcp://127.0.0.1:28331

# The RPC commands below are custom, you can change it to whatever
rpcallowip=127.0.0.1
rpcuser=bitcoin
rpcpassword=local321
rpcport=18447

# I don't believe this line is needed, but I just leave it
uacomment=bitcore
```

Once everything is setup, you can simply start the node with (while in the folder `mynode-abc`)
```
bitcore start
```
You can also access the the User Interface on `localhost:[port-from-config]/`

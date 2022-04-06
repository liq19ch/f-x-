# f-x-
## Set up a node validator with Windows
A doc for each step to set up a node validator with f(x).

## Techonologies
* git bash (or ubuntu)

## Setup preparation
*gcc & make:

download tdm64-gcc-10.3.0-2 from https://jmeubank.github.io/tdm-gcc/articles/2021-05/10.3.0-release

download chocolate for downloading make https://chocolatey.org/install
Run:
```
$ choco install make
```
Have the version 10.3.0 for gcc & 4.3 for make
![gcc-version]()
![make-version]()

*go:
```
$ wget https://dl.google.com/go/go1.17.7.linux-amd64.tar.gz 
$ mkdir -p $HOME/go/bin
$ echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
$ echo "export PATH=$PATH:$(go env GOPATH)/bin" >> ~/.profile
$ 10.3.0
```

**notice:! if you are using ubuntu, this will not work. Since ubuntu only support you with the go version up to 1.13, you'll need to find another way to set up the path with this.
**notice:! a windows user may need to download wget command first.


*binaries
```
git clone https://github.com/functionx/fx-core.git
cd fx-core
make go.sum
```

clone the repo and cd into fx-core to make go.sum.

swith to testnet env to intall:

```
git checkout testnet-evm
make go.sum
make install-testnet
```

*check network
```
fxcored network
```

![network]()

*check version
```
fxcored version
```

![fxcored-version]()


## Setup full node
*init
```
fxcored init fx-zakir
```

*overwrite the toml files line by line
```
wget https://raw.githubusercontent.com/functionx/fx-core/testnet-evm/public/testnet/genesis.json -O ~/.fxcore/config/genesis.json
wget https://raw.githubusercontent.com/functionx/fx-core/testnet-evm/public/testnet/config.toml -O ~/.fxcore/config/config.toml
wget https://raw.githubusercontent.com/functionx/fx-core/testnet-evm/public/testnet/app.toml -O ~/.fxcore/config/app.toml
```

*snapshot
snapshot will be performed every Monday morning at 2:00 am UTC. So the date is needed to be changed to the Monday you are going to download it.
This is a large zip file which may take whole day to be downloaded. Make sure the network is stable and check the download process randomly.
If the process stopped, just ctrl+c to stop it and rerun the same command line below. wget is smart enought to download it continuously. 

```
wget -c https://fx-testnet.s3.amazonaws.com/fxcore-snapshot-testnet-2022-03-28.tar.gz
```

unpack it:

```
tar -xzvf fxcore-snapshot-testnet-2022-03-28.tar.gz -C ~/.fxcore/
```

start a node:
```
nohup fxcored start 2>&1 > fxcore.log &
```

## Create a validator
*a new validator name
```
fxcored keys add <_name>
```

**notice:! a full node can only be bound to one validator.

use the command to ensure it's all sync:
```
fxcored status
```
![status]()

*take some FX from faucet
choose the first one when you are trying to create a validator in testnet.
https://dhobyghaut-faucet.functionx.io/

![faucet]()

*ensure the path
results should be the same.
```
curl -s 127.0.0.1:26657/status | jq '.result.validator_info.pub_key'
cat .fxcore/config/priv_validator_key.json| jq .pub_key
```

the last second should be the same with the second command:
```
fxcored keys parse $(cat .fxcore/config/priv_validator_key.json| jq -r '.address')
fxcored tendermint show-address
```


**notice:! a windows users may need to download jq command.


*setup the validator with a transaction
the amount should be over your min-self-delegation, otherwise it will fail.

```
fxcored tx staking create-validator \
  --chain-id=dhobyghaut \
  --gas="auto" \
  --gas-adjustment=1.2 \
  --gas-prices="4000000000000FX" \
  --from=<_name> \
  --amount=500000000000000000000FX \
  --pubkey=$(fxcored tendermint show-validator) \
  --commission-rate="0.01" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="100000000000000000000" \
  --moniker="choose a moniker" \
  --website="https://functionx.io" \
  --details="To infinity and beyond!" 
  ```
  
  *search yours with explorer
  ensure it's successful.
  https://dhobyghaut-explorer.functionx.io/
  

my validator creation:
https://dhobyghaut-explorer.functionx.io/fxcore/tx/0xB53C2F580F2BB96782363EB350EE4B46F87DCE01E76910CA9CE5E46C4088088C

# Deployment guide for xdai bridge validator

## XDAI Bridge Validator Deployment Information

This document contains the information to deploy a Validator for the XDAI Bridge in testing environments (chiado-goerli).

In order to have a better debugging experience, we will use different contracts.

## Contracts

The following contracts are deployed and verified in their respectives networks (home -> chiado, foreign -> goerli):

- [Home] Bridge Validators Proxy: [0x81Fa3a52bf1D8B3F10F9b70793c3193c22C8cc62](https://blockscout.chiadochain.net/address/0x81Fa3a52bf1D8B3F10F9b70793c3193c22C8cc62)
- [Home] Bridge Validators Impl: [0x3Fd5a613c0432e98AB6Fb6D53633a22c49fD589B](https://blockscout.chiadochain.net/address/0x3Fd5a613c0432e98AB6Fb6D53633a22c49fD589B)
- [Home] BlockRewardMock: [0xfFc23F15Cc5CC5255093b7Ce1923D1395A2F70a1](https://blockscout.chiadochain.net/address/0xfFc23F15Cc5CC5255093b7Ce1923D1395A2F70a1)
- [Home] Home Bridge Proxy: [0x1558B3D83D56bDF55Db1E9410AAdBD2F79057228](https://blockscout.chiadochain.net/address/0x1558B3D83D56bDF55Db1E9410AAdBD2F79057228)
- [Home] Home Bridge Impl: [0x4224525BFF275033A7396ad5A18d49947162b1E1](https://blockscout.chiadochain.net/address/0x4224525BFF275033A7396ad5A18d49947162b1E1)
- [Foreign] BridgeValidators Storage:  [0x8ac414768a281f75d3f8542dbc7f0dd5df240dd9](https://goerli.etherscan.io/address/0x8ac414768a281f75d3f8542dbc7f0dd5df240dd9)
- [Foreign] BridgeValidators Implementation:  [0x52a181339857d8291ad3b0c6c909f6b5cde51665](https://goerli.etherscan.io/address/0x52a181339857d8291ad3b0c6c909f6b5cde51665)
- [Foreign] ForeignBridge Proxy: [0x71cf72bdc6b3f77a12c7521e34c711c7dec0c336](https://goerli.etherscan.io/address/0x71cf72bdc6b3f77a12c7521e34c711c7dec0c336)
- [Foreign] ForeignBridge Impl: [0x19cd330ddeced57469de9527864880a07395c739](https://goerli.etherscan.io/address/0x19cd330ddeced57469de9527864880a07395c739)
- [Foreign] Dai: [0xdc31ee1784292379fbb2964b3b9c4124d8f89c60](https://goerli.etherscan.io/address/0xdc31ee1784292379fbb2964b3b9c4124d8f89c60)


## Deployment

In this example we will deploy a bridge validator in an AWS EC2 instance.

### 1) Create an AWS EC2 Instance

Example

![1-aws-instance](https://user-images.githubusercontent.com/13955827/201762562-64a72409-01f7-489b-82b2-86e477c77ace.png)


Configure a keypair as we will use a ssh connection to connect to the VM.

Remember the minimum requirements:
- 1 CPU, 4Gb RAM, 50Gb Disk


Note: 

Follow best security practices, bridge validator doesn't require any open ports

you can use tips from [here](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node#setup-two-factor-authentication-for-ssh-optional)


### 2) Install repositories

Download and replace the deployment folder

- git clone --recursive https://github.com/omni/tokenbridge
- git clone https://github.com/gnosischain/chiado-ansible-bridges.git
- rm -r tokenbridge/deployment/
- mv chiado-ansible-bridges/ tokenbridge/deployment/

### 3) Configure Host ssh connection

Configure the following variables in the hosts.yml:
- hosts: Instance IP
- ansible_ssh_private_key_file: path to the .pem keypair file
- ORACLE_VALIDATOR_ADDRESS_PRIVATE_KEY: private key of the validator to be used (you can use ansible-vault to secure this var)

![2-host-config-example](https://user-images.githubusercontent.com/13955827/201762659-31f0ad16-8305-484f-8c19-b7138eb3940e.png)

### 4) Configure XDAI contracts (Native)

Edit the following file: `tokenbridge/deployment/group_vars/native.yml` in order to use the already verified contracts:

```
COMMON_HOME_BRIDGE_ADDRESS: 0x1558B3D83D56bDF55Db1E9410AAdBD2F79057228
COMMON_FOREIGN_BRIDGE_ADDRESS: 0x71cf72bdc6b3f77a12c7521e34c711c7dec0c336
```

### 5) Run Ansible Playbook (before that please check the monitoring sectioon below)

- ansible-playbook -e key=/path/to/public_key.pub -i hosts.yml site.yml --limit instance_ip

![3-install-example](https://user-images.githubusercontent.com/13955827/201762808-d219d880-b660-4970-8186-e6fa8e9eb104.png)

### 6) Verify processes running

![4-containers-running](https://user-images.githubusercontent.com/13955827/201763263-fca35b2d-7357-4f2a-b10e-4c3bd062f7e3.png)


## Test

Before testing bridges ensure the following steps:

### Allow DAI Contract

User Account must call approve in the DAI contract to the ForeignBridge Proxy address (unlimited amount: 999999999999900000000000000000)
- DAI: 0xdc31ee1784292379fbb2964b3b9c4124d8f89c60
- ForeignBridge: 0x71cf72bdc6b3f77a12c7521e34c711c7dec0c336
- calldata example: "0x095ea7b300000000000000000000000071cf72bdc6b3f77a12c7521e34c711c7dec0c336000000000000000000000000000000000000000c9f2c9cd04511a871e2760000"

  How-to:

  Go to DAI contract on goerli: [DAI](https://goerli.etherscan.io/address/0xdc31ee1784292379fbb2964b3b9c4124d8f89c60)

  Go to mycrypto -> tools -> contracts , [here](https://app.mycrypto.com/interact-with-contracts)

  Choose network, paste contract adress, and then paste any ERC20 token ABI [for ex:](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code)

  Then interact and write approve method with given [calldata](https://rimeissner.dev/transaction-decoder/#/?data=0x095ea7b300000000000000000000000071cf72bdc6b3f77a12c7521e34c711c7dec0c336000000000000000000000000000000000000000c9f2c9cd04511a871e2760000)

  Connect wallet and sign the tx

### BlockReward contract needs balance to reward

Verify that [**BlockRewardMock**](https://blockscout.chiadochain.net/address/0xfFc23F15Cc5CC5255093b7Ce1923D1395A2F70a1) contract has enough XDAI balance to reward/transfer (minimum bridging amount).

### Stop bridge processes (Production)

- In order to match the validators in production (user must claim tokens manually)

the following services must be turned off:
- bridge_senderforeign
- bridge_collected

## Bridge Examples

UI: https://spectacular-churros-a1e6a5.netlify.app/

### goerli -> chiado

- https://goerli.etherscan.io/tx/0xa883756aebf4a94bbea635d8a3a0431e4205a5b28c6083ff802d0d06466fec9f
- https://blockscout.chiadochain.net/tx/0xe3c1927e9f2c0b55005b3870777b264439e59b53b25161e2fa7f268e4b0e72a3

### chiado -> goerli

- https://blockscout.chiadochain.net/tx/0xc271c6765b11dfa71ace5ec0db8221bffacda6a44154eb1693bf5d9f97da79df
- https://blockscout.chiadochain.net/tx/0x7117605e32486ee8348e621c7146ae421a40a3328fd7f70d9a2b1d4e8159203b
- https://goerli.etherscan.io/tx/0x0d0a8f359746d8f7f35a553d8d87ccf2b0fa48bc955719aa9470c7ae7cfe79ae


## Monitoring and Alerting stack

We have added monitoring+alering stack with Prometheus, Grafana and Loki

- Please go to the `Monitoring` folder and edit docker-compose.yml as per the need

- You Should change the `Grafana` ENVs and `Traefik` Labels

- Go to `alertmanager` folder and replace the `api_url` with your slack webhook ( This will send you alerts )

- Go to `prometheus` folder and change the alerts rule as per the need

## Traefik Proxy

We have added traefik as a proxy! Grafana will be accessibe using given domain name!

- You should have given the labels to grafana in above steps

- Go to `traefik` folder and modify docker-compose with needed parameters~ your email-id for LetsEncrypt

Notes: 

- You should make a DNS entry with given hostname to the server's ip

- In order to get the acme-challenge verfied against the domain, the ports 80/443 should be opened for a while

- open ports 80/443 in your filewall for sometime, go to server

  ```shell
  cd /home/poadocker/bridge/traefik/letsencrypt
  cat acme.json
  ```

  You should see the cert has been added, if its there you can hit the domain and check

  Do not forget to close ports for public access (You should whitelist the ips to access 80/443)

- Also IP:3000 will lead to the grafana, just in case if you don't want to use proxy!

## PGL stack usecase

- With prometheus/grafana you should have a better monitoring and alering

- With loki , you should be able to see the logs of all the containers

- Visit `grafana->explorer` , select loki as data source and watch out for logs

- `Node-exporter` dashbord will give you system level metrics!


## Troubleshoot

If you are not able to access grafana using domain name

- check the traefik logs

- Restart the traefik container if the cert is already present

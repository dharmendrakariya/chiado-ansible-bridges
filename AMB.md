# Deployment guide for AMB bridge validator

## AMB Bridge Validator Deployment Information

This document contains the information to deploy a Validator for the AMB Bridge in testing environments (chiado-goerli).

This will follow the documentation from this repository: https://github.com/gnosischain/chiado-ansible-bridges.

In order to have a better debugging experience, we will use different contracts.

Also, we will use omnibridge mediator contracts in order to test the underlying amb bridge.

## Contracts

The following contracts are deployed and verified in their respectives networks (home -> chiado, foreign -> goerli):

```
[Foreign] BridgeValidators Proxy: 0x8a554153AF784d55cFf0378e7289b492C2f61B21
[Foreign] BridgeValidators Impl: 0x924033562106673C4dc01B58c4F2a9ff2977B1bA

[Foreign] ForeignBridgeAMB Proxy: 0xa3264bAFa607BDe904A9A400d2C3a3a3d0C79eA8
[Foreign] ForeignBridgeAMB Impl: 0x3A21f51c2e553bB2aF886a6d16586CD8C991EaBA

[Foreign] PermittableToken: 0x961df79e1dbc3712091a652ce10d50f7c8e62f28
[Foreign] TokenFactory: 0xba4adc73208ca222d08be9772e7ed79a63ff9231

[Foreign] ForeignOmnibridge Proxy: 0x8281f970D4a7D5EC75c9F1cfA75D29d6a1d878b7
[Foreign] ForeignOmnibridge Impl: 0x1b73607773d141708459bfb1eb0e702fc4582b52

[Home] BridgeValidators Proxy: 0xdA3614E6576E4E91d5Df43C857c5c84A8ab26e7A
[Home] BridgeValidators Impl: 0xb294E10B39c530c97CFC0424Fe7C5f025Cc3b84E

[Home] HomeBridgeAMB Proxy: 0x60b0f8800039457de96D1b6c0A13fc270F1C99CE
[Home] HomeBridgeAMB Impl: 0x266185A479f07F9919571E32eBB3F8419B762650

[Home] HomeOmnibridge Proxy: 0x3C24710c7bDEf5cb54ABf28595DD9c4b05730ef9
[Home] HomeOmnibridge Impl: 0x13EEfAB0d737Dfa03828140CaD226BBbe6474c8D

[Home] PermittableToken: 0x3c16E8BA62A66394CED34b5Ccf7525A46B6df922
[Home] TokenFactory: 0x89E5749729324fd0f834abFe6E251719Ef47c77b (unverified)

[Home] OmnibridgeFeeManager: 0x0000000000000000000000000000000000000000
[Home] ForwardingRulesManager: 0x0000000000000000000000000000000000000000
[Home] SelectorTokenGasLimitManager: 0x751340B8e7c6a68dd209f725c2D8bd5435C5fDA3
```
## Deployment

In this example we will deploy a bridge validator in an AWS EC2 instance.

### 1) Create an AWS EC2 Instance

Example

![1-amb-aws-instance](https://user-images.githubusercontent.com/13955827/201764306-be8cec73-bbcc-42aa-8987-a46d1a1914a0.png)


Configure a keypair as we will use a ssh connection to connect to the VM

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

### 4) Configure AMB contracts (Native)

Edit the following file: `tokenbridge/deployment/group_vars/amb.yml` in order to use the already verified contracts:

```
COMMON_HOME_BRIDGE_ADDRESS: 0x60b0f8800039457de96D1b6c0A13fc270F1C99CE
COMMON_FOREIGN_BRIDGE_ADDRESS: 0xa3264bAFa607BDe904A9A400d2C3a3a3d0C79eA8
```

### 5) Run Ansible Playbook

- ansible-playbook -e key=/path/to/public_key.pub -i hosts.yml site.yml --limit amb_instance_ip

![3-install-example](https://user-images.githubusercontent.com/13955827/201762808-d219d880-b660-4970-8186-e6fa8e9eb104.png)

### 6) Verify processes running

![4-containers-running-amb](https://user-images.githubusercontent.com/13955827/201766822-dc7e9b9f-8527-47b6-b09a-2e54b3d624be.png)

## Bridge Examples

UI: https://wondrous-bunny-fefb60.netlify.app/

goerli -> chiado
- https://goerli.etherscan.io/tx/0x10c881da00e7b1d2acd7d271f9b2a3a192618f11f822c0d65c14752d8dea3c7f
- https://blockscout.chiadochain.net/tx/0x5eaaf9e1d3ccc418ddfabcae6625d25b2ac627e76719d283f5bf0f394890f842

chiado -> goerli
- https://blockscout.chiadochain.net/tx/0x96910aaf8550eb7467ac2a960942feb4eed5cf8843fee8db718cda687fa039e9
- https://blockscout.chiadochain.net/tx/0x9a76237748e427f49bdf598005981fd06f3eac87d39a47d5b003304e0c7d8ed8
- https://goerli.etherscan.io/tx/0x8e0ee62dfca9bcae6f1ce759370df93e04f8e72da008a1744a7195dedee520f4


### Stop bridge processes (Production)

- In order to match the validators in production (user must claim tokens manually)

the following services must be turned off:
- bridge_senderforeign
- bridge_collected


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


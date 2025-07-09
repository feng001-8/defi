## ETH Staking 

![](_img/eth-stake.png)

### example:

let's say that Alice has 32 ETHs. With her 32 ETH, she wants to run a validator and secure the ETH network and earn some ETH rewards ETH. The first step is to stake her 32 ETH. She does this by sending a transaction to a deposit contract, sending her 32 ETH, and the validator public key which is derived from her validator key,and her withdrawal credentials which is derived from her withdrawal key. The withdrawal key is later used when she wants to unstake her 32 ETH. Next she will run a validator.The validator key is used by the validator to sign blocks and attest to transactions. So she'll need to take this validator key, and then stick it inside this server that runs the validator software.

## Rocket Pool

![](_img/rocketPool.png)

### example:

The way it works is that all three of these users will interact with the RocketPool contracts.Alice will deposit 1 ETH, Linus will deposit 8 ETH, and also Alice register as a node operator. Now remember that to become a validator in the ETH network, you need a multiple of 32 ETH.And now Linus can register as a validator within the RocketPool protocol to become a validator.Now here, instead of running the ETH validator software, Linus will have to run a RocketPool software. This RocketPool software will interact with the ETH network to earn ETH staking rewards.



## Rebase Token and non-Rebase Token

![](_img/rebaseToken.png)

### example:

* rebase token: 余额随着价值变动
* non-rebase token：余额不会随着价值变动
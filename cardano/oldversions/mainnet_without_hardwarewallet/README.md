# Description - StakepoolOperator Scripts - Mainnet Allegra Era

## First of all, you don't need them all! Examples are on this page :smiley:

:bulb: **FOR USE WITH CARDANO-NODE & CARDANO-CLI: tags/1.24.2 !  (git checkout tags/1.24.2)**

:bulb: **PLEASE USE THE CONFIG AND GENESIS FILES FROM [here](https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html) -> mainnet**

Theses scripts here should help you to manage your StakePool via the CLI. As always use them at your own risk, but they should be errorfree. Scripts were made to make things easier while learning all the commands and steps to bring up the stakepool node. So, don't be mad at me if something is not working. CLI calls are different almost daily currently.<br>
Some scripts are using **jq, curl & bc** so make sure you have it installed ```(sudo apt update && sudo apt install -y jq curl bc)```

Contacts: Telegram - [@atada_stakepool](https://t.me/atada_stakepool), Twitter - [@ATADA_Stakepool](https://twitter.com/ATADA_Stakepool), Homepage - https://stakepool.at 

If you can't hold back and wanna give me a little Tip, here's my MainNet Shelley Ada Address, thx! :-)
```addr1q9vlwp87xnzwywfamwf0xc33e0mqc9ncznm3x5xqnx4qtelejwuh8k0n08tw8vnncxs5e0kustmwenpgzx92ee9crhxqvprhql```


# Online-Mode vs. Offline-Mode

The scripts are capable to be used in Online- and Offline-Mode. It depends on your setup, your needs and just how you wanna work. Doing transactions with pledge accounts in Online-Mode can be a security risk, also doing Stakepool-Registrations in pure Online-Mode can be risky. To enhance Security the scripts can be used on a Online-Machine and an Offline-Machine. You only have to **transfer one single file (offlineTransfer.json)** between the Machines. If you wanna use the Offline-Mode, your **Gateway-Script** to get Data In/Out of the offlineTransfer.json is the **01_workOffline.sh** Script. The **offlineTransfer.json** is your carry bag between the Machines.<br>

Why not always using Offline-Mode? You have to do transactions online, you have to check balances online. Also, there are plenty of usecases using small wallets without the need of the additional steps to do all offline everytime. Also if you're testing some things on Testnets, it would be a pain to always transfer files between the Hot- and the Cold-Machine. You choose how you wanna work... :-)<br>

<details>
   <summary>How do you switch between Online- and Offline-Mode?</summary>
   
<br>Thats simple, you just change a single entry in the 00_common.sh, common.inc or $HOME/.common.inc config-file:
<br>```offlineMode="no"``` Scripts are working in Online-Mode
<br>```offlineMode="yes"``` Scripts are working in Offline-Mode

So on the Online-Machine you set the ```offlineMode="no"``` and on the Offline-Machine you set the ```offlineMode="yes"```.

</details>

<details>
   <summary>What do you need on the Online- and the Offline-Machine?</summary>
   
<br>On the Online-Machine you need a running and fully synced cardano-node, the cardano-cli and also your ```*.addr``` files to query the current balance of them for the Offline-Machine. **You should not have any signing keys ```*.skey``` files of big wallets laying around!** Metadata-Files are fine, you need them anyway to transfer them to your Stakepool-Webserver, also they are public available, no security issue.

On the Offline-Machine you have your signing keys, thats the ```*.skey``` files, also you have your kes-keys, vrf-keys, opcerts, etc. on this Machine.
You need the cardano-cli on the Offline-Machine, same version as on the Online-Machine! You don't need the cardano-node, because you will never be online with that Machine!

You should keep your directory structure the same on both Machines.

:bulb: **Best practice Advise for Stakepool Operators:** Even that the Offline-Machine is pretty secure, it can be compromised by a physical attack on in. So, after you   have registered your Pool, move away our pledge payment signing keys like the ```owner.payment.skey``` from the machine and store it in a secure place. You don't need it to update your Stakepool. To do some updates, generate yourself a few small payment wallets with the script 02. To do more than one transaction in a single "Online->Offline->Online" process i would recommend to have at least three small wallets with a few ADA on it. You can do all Pool-Updates, Re-Registrations, Reward-Claims, etc. with theses.<br>**There is no need to have the pledge owner payment signing keys on that machine at all!**

</details>

# Scriptfiles Syntax

<details>
   <summary>Show the full Syntax details for each script...</summary>


* **00_common.sh:** main config file (!) set your variables in there for your config, will be used by the scripts.<br>
  
  **Overwritting the default settings:** You can now place a file with name ```common.inc``` in the calling directory and it will be sourced by the 00_common.sh automatically. So you can overwrite the setting-variables dynamically if you want. Or if you wanna place it in a more permanent place, you can name it ```.common.inc``` and place it in the user home directory. The ```common.inc``` in a calling directory will overwrite the one in the home directory if present. <br>
  :bulb: You can also use it to set the CARDANO_NODE_SOCKET_PATH environment variable by just calling ```source ./00_common.sh```

* **01_workOffline.sh:** this is the script you're doing your **Online**->**Offline**->**Online**->**Offline** work with
<br>```./01_workOffline.sh <command> [additional data]``` 
<br>```./01_workOffline.sh new``` Resets the offlineTransfer.json with only the current protocol-parameters in it (OnlineMode only)
<br>```./01_workOffline.sh info``` Displayes the Address and TX info in the offlineTransfer.json<br>
<br>```./01_workOffline.sh add mywallet``` Adds the UTXO info of mywallet.addr to the offlineTransfer.json (OnlineMode only)
<br>```./01_workOffline.sh add owner.staking``` Adds the Rewards info of owner.staking to the offlineTransfer.json (OnlineMode only)<br>
<br>```./01_workOffline.sh execute``` Executes the first cued transaction in the offlineTransfer.json (OnlineMode only)
<br>```./01_workOffline.sh execute 3``` Executes the third cued transaction in the offlineTransfer.json (OnlineMode only)<br>
<br>```./01_workOffline.sh attach <filename>``` This will attach a small file (filename) into the offlineTransfer.json
<br>```./01_workOffline.sh extract``` Extract the attached files in the offlineTransfer.json<br>
<br>```./01_workOffline.sh cleartx``` Removes the cued transactions in the offlineTransfer.json
<br>```./01_workOffline.sh clearhistory``` Removes the history in the offlineTransfer.json
<br>```./01_workOffline.sh clearfiles``` Removes the attached files in the offlineTransfer.json

The scripts uses per default (configurable) the file **offlineTransfer.json** to store the data in between the Machines.

* **01_queryAddress.sh:** checks the amount of lovelaces and tokens on an address with autoselection about a UTXO query on enterprise & payment(base) addresses or a rewards query for stake addresses
<br>```./01_queryAddress.sh <name or hash>``` **NEW** you can use the HASH of an address too now.
<br>```./01_queryAddress.sh addr1``` shows the lovelaces from addr1.addr
<br>```./01_queryAddress.sh owner.staking``` shows the current rewards on the owner.staking.addr
<br>```./01_queryAddress.sh addr1vyjz4gde3aqw7e2vgg6ftdu687pcnpyzal8ax37cjukq5fg3ng25m``` shows the lovelaces on this given Bech32 address
<br>```./01_queryAddress.sh stake1u9w60cpjg0xnp6uje8v3plcsmmrlv3vndcz0t2lgjma0segm2x9gk``` shows the rewards on this given Bech32 address

* **01_sendLovelaces.sh:** sends a given amount of lovelaces or ALL lovelaces or ALLFUNDS lovelaces+tokens from one address to another, uses always all UTXOs of the source address
<br>```./01_sendLovelaces.sh <fromAddr> <toAddrName or hash> <lovelaces>``` (you can send to an HASH address too)
<br>```./01_sendLovelaces.sh addr1 addr2 1000000``` to send 1000000 lovelaces from addr1.addr to addr2.addr
<br>```./01_sendLovelaces.sh addr1 addr2 ALL``` to send **ALL lovelaces** from addr1.addr to addr2.addr, Tokens on addr1.addr are preserved
<br>```./01_sendLovelaces.sh addr1 addr2 ALLFUNDS``` to send **ALL funds** from addr1.addr to addr2.addr **including Tokens** if present
<br>```./01_sendLovelaces.sh addr1 addr1vyjz4gde3aqw7e2vgg6ftdu687pcnpyzal8ax37cjukq5fg3ng25m ALL``` send ALL lovelaces from addr1.addr to the given Bech32 address

* **01_sendAssets.sh:** sends Assets(Tokens) and optional a given amount of lovelaces from one address to another
<br>```./01_sendAssets.sh <fromAddr> <toAddress|HASH> <PolicyID.Name|<PATHtoNAME>.asset> <AmountOfAssets|ALL> [Optional Amount of lovelaces to attach]```
<br>```./01_sendAssets.sh addr1 addr2 mypolicy.SUPERTOKEN 15```<br>to send 15 SUPERTOKEN from addr1.addr to addr2.addr with minimum lovelaces attached
<br>```./01_sendAssets.sh addr1 addr2 mypolicy.MEGATOKEN ALL 12000000```<br>to send **ALL** MEGATOKEN from addr1.addr to addr2.addr and also 12 ADA
<br>```./01_sendAssets.sh addr1 addr2 34250edd1e9836f5378702fbf9416b709bc140e04f668cc355208518.ATADAcoin 120```<br>to send 120 Tokens of Type 34250edd1e9836f5378702fbf9416b709bc140e04f668cc355208518.ATADAcoin from addr1.addr to addr2.addr. Using the PolicyID.TokenNameHASH allowes you to send out Tokens you've got from others. You own generated Tokens can be referenced by the policyName.tokenName schema.

* **01_claimRewards.sh:** claims all rewards from the given stake address and sends it to a receiver address
<br>```./01_claimRewards.sh <nameOfStakeAddr> <toAddr> [optional <feePaymentAddr>]```
<br>```./01_claimRewards.sh owner.staking owner.payment``` sends the rewards from owner.staking.addr to the owner.payment.addr. The transaction fees will also be paid from the owner.payment.addr
<br>```./01_claimRewards.sh owner.staking myrewards myfunds``` sends the rewards from owner.staking.addr to the myrewards.addr. The transaction fees will be paid from the myfunds.addr

* **01_sendVoteMeta.sh:** modified sendLoveLaces script to simply send a voting json metadata file
<br>```./01_sendLovelaces.sh <fromAddr> <VoteFileName>```
<br>```./01_sendLovelaces.sh addr1 myvote``` to just send the myvote.json votingfile from funds on addr1.addr
<br>Also please check the Step-by-Step notes [HERE](#bulb-how-to-do-a-voting-for-spocra-in-a-simple-process)

* **02_genPaymentAddrOnly.sh:** generates an "enterprise" address with the given name for just transfering funds
<br>```./02_genPaymentAddrOnly.sh <name>```
<br>```./02_genPaymentAddrOnly.sh addr1``` will generate the files addr1.addr, addr1.skey, addr1.vkey<br>

* **03a_genStakingPaymentAddr.sh:** generates the base/payment address & staking address combo with the given name and also the stake address registration certificate
<br>```./03a_genStakingPaymentAddr.sh <name>```
<br>```./03a_genStakingPaymentAddr.sh owner``` will generate the files owner.payment.addr, owner.payment.skey, owner.payment.vkey, owner.staking.addr, owner.staking.skey, owner.staking.vkey, owner.staking.cert<br>

* **03b_regStakingAddrCert.sh:** register the staking address on the blockchain with the certificate from 03a.
<br>```./03b_regStakingAddrCert.sh <nameOfStakeAddr> <nameOfPaymentAddr>```
<br>```./03b_regStakingAddrCert.sh owner.staking addr1``` will register the staking addr owner.staking using the owner.staking.cert with funds from addr1 on the blockchain. you could of course also use the owner.payment address here for funding.<br>

* **03c_checkStakingAddrOnChain.sh:** check the blockchain about the staking address
<br>```./03c_checkStakingAddrOnChain.sh <name>```
<br>```./03c_checkStakingAddrOnChain.sh owner``` will check if the address in owner.staking.addr is currently registered on the blockchain

* **04a_genNodeKeys.sh:** generates the poolname.node.vkey and poolname.node.skey cold keys and resets the poolname.node.counter file
<br>```./04a_genNodeKeys.sh <poolname>```
<br>```./04a_genNodeKeys.sh mypool```

* **04b_genVRFKeys.sh:** generates the poolname.vrf.vkey/skey files
<br>```./04b_genVRFKeys.sh <poolname>```
<br>```./04b_genVRFKeys.sh mypool```

* **04c_genKESKeys.sh:** generates a new pair of poolname.kes-xxx.vkey/skey files, and updates the poolname.kes.counter file. every time you generate a new keypair the number(xxx) autoincrements. To renew your kes/opcert before the keys of your node expires just rerun 04c and 04d!
<br>```./04c_genKESKeys.sh <poolname>```
<br>```./04c_genKESKeys.sh mypool```

* **04d_genNodeOpCert.sh:** calculates the current KES period from the genesis.json and issues a new poolname.node-xxx.opcert certificate. it also generates the poolname.kes-expire.json file which contains the valid start KES-Period and also contains infos when the generated kes-keys will expire. to renew your kes/opcert before the keys of your node expires just rerun 04c and 04d! after that, update the files on your stakepool server and restart the coreNode
<br>```./04d_genNodeOpCert.sh <poolname>```
<br>```./04d_genNodeOpCert.sh mypool```

* **05a_genStakepoolCert.sh:** generates the certificate poolname.pool.cert to (re)register a stakepool on the blockchain
  <br>```./05a_genStakepoolCert.sh <PoolNodeName> [optional registration-protection key]``` will generate the certificate poolname.pool.cert from poolname.pool.json file<br>
  To register a protected Ticker you will have to provide the secret protection key as a second parameter to the script.<br>
  The script requires a json file for the values of PoolNodeName, OwnerStakeAddressName(s), RewardsStakeAddressName (can be the same as the OwnerStakeAddressName), pledge, poolCost & poolMargin(0.01-1.00) and PoolMetaData. This script will also generate the poolname.metadata.json file for the upload to your webserver:
  <br>**Sample mypool.pool.json**
  ```console
   {
      "poolName": "mypool",
      "poolOwner": [
         {
         "ownerName": "owner",
         "ownerWitness": "local"
         }
      ],
      "poolRewards": "owner",
      "poolPledge": "100000000000",
      "poolCost": "500000000",
      "poolMargin": "0.10",
      "poolRelays": [
         {
         "relayType": "ip or dns",
         "relayEntry": "x.x.x.x or the dns-name of your relay",
         "relayPort": "3001"
         }
      ],
      "poolMetaName": "This is my Pool",
      "poolMetaDescription": "This is the description of my Pool!",
      "poolMetaTicker": "POOL",
      "poolMetaHomepage": "https://mypool.com",
      "poolMetaUrl": "https://mypool.com/mypool.metadata.json",
      "poolExtendedMetaUrl": "",
      "---": "--- DO NOT EDIT BELOW THIS LINE ---"
    }
   ```
   :bulb:   **If the json file does not exist with that name, the script will generate one for you, so you can easily edit it.**<br>

   poolName is the name of your poolFiles from steps 04a-04d, poolOwner is an array of all the ownerStake from steps 03, poolRewards is the name of the stakeaddress getting the pool rewards (can be the same as poolOwner account), poolPledge in lovelaces, poolCost per epoch in lovelaces, poolMargin in 0.00-1.00 (0-100%).<br>
   poolRelays is an array of your IPv4/IPv6 or DNS named public pool relays. Currently the types DNS, IP, IP4, IPv4, IP6 and IPv6 are supported. Examples of multiple relays can be found [HERE](#using-multiple-relays-in-your-poolnamepooljson) <br> MetaName/Description/Ticker/Homepage is your Metadata for your Pool. The script generates the poolname.metadata.json for you. In poolMetaUrl you must specify your location of the file later on your webserver (you have to upload it to this location). <br>There is also the option to provide ITN-Witness data in an extended metadata json file. Please read some infos about that [here](#bulb-itn-witness-ticker-check-for-wallets)<br>After the edit, rerun the script with the name again.<br>
   > :bulb:   **Update Pool values (re-registration):** If you have already registered a stakepool on the chain and want to change some parameters, simply [change](#file-autolock) them in the json file and rerun the script again. The 05c_regStakepoolCert.sh script will later do a re-registration instead of a new registration for you.

* **05b_genDelegationCert.sh:** generates the delegation certificate name.deleg.cert to delegate a stakeAddress to a Pool poolname.node.vkey. As pool owner you have to delegate to your own pool, this is registered as pledged stake on your pool.
<br>```./05b_genDelegationCert.sh <PoolNodeName> <DelegatorStakeAddressName>```
<br>```./05b_genDelegationCert.sh mypool owner``` this will delegate the Stake in the PaymentAddress of the Payment/Stake combo with name owner to the pool mypool

* **05c_regStakepoolCert.sh:** (re)register your **poolname.pool.cert certificate** and also the **owner name.deleg.cert certificate** with funds from the given name.addr on the blockchain. it also updates the pool-ID and the registration date in the poolname.pool.json
<br>```./05c_regStakepoolCert.sh <PoolNodeName> <PaymentAddrForRegistration> [optional REG / REREG keyword]```
<br>```./05c_regStakepoolCert.sh mypool owner.payment``` this will register your pool mypool with the cert and json generated with script 05a on the blockchain. Owner.payment.addr will pay for the fees.<br>
If the pool was registered before (when there is a **regSubmitted** value in the name.pool.json file), the script will automatically do a re-registration instead of a registration. The difference is that you don't have to pay additional fees for a re-registration.<br>
  > :bulb: If something went wrong with the original pool registration, you can force the script to redo a normal registration by adding the keyword REG on the commandline like ```./05c_regStakepoolCert.sh mypool mywallet REG```<br>
Also you can force the script to do a re-registration by adding the keyword REREG on the command line like ```./05c_regStakepoolCert.sh mypool mywallet REREG```

* **06_regDelegationCert.sh:** register a simple delegation (from 05b) name.deleg.cert 
<br>```./06_regDelegationCert.sh <delegatorName> <nameOfPaymentAddr>```
<br>```./06_regDelegationCert.sh someone someone.payment``` this will register the delegation certificate someone.deleg.cert for the stake-address someone.staking.addr on the blockchain. The transaction fees will be paid from someone.payment.addr.

* **07a_genStakepoolRetireCert.sh:** generates the de-registration certificate poolname.pool.dereg-cert to retire a stakepool from the blockchain
  <br>```./07a_genStakepoolRetireCert.sh <PoolNodeName> [optional retire EPOCH]```
  <br>```./07a_genStakepoolRetireCert.sh mypool``` generates the mypool.pool.dereg-cert to retire the pool in the NEXT epoch
  <br>```./07a_genStakepoolRetireCert.sh mypool 253``` generates the poolname.pool.dereg-cert to retire the pool in epoch 253<br>
  The script requires a poolname.pool.json file with values for at least the PoolNodeName & OwnerStakeAddressName. It is the same json file we're already using since script 05a, so a total pool history json file.<br>
  **If the json file does not exist with that name, the script will generate one for you, so you can easily edit it.**<br>
   poolName is the name of your poolFiles from steps 04a-04d, poolOwner is the name of the StakeOwner from steps 03

* **07b_deregStakepoolCert.sh:** de-register (retire) your pool with the **poolname.pool.dereg-cert certificate** with funds from name.payment.addr from the blockchain. it also updates the de-registration date in the poolname.pool.json
<br>```./07b_deregStakepoolCert.sh <PoolNodeName> <PaymentAddrForDeRegistration>```
<br>```./07b_deregStakepoolCert.sh mypool mywallet``` this will retire your pool mypool with the cert generated with script 07a from the blockchain. The transactions fees will be paid from the mywallet.addr account.<br>
  :bulb: Don't de-register your rewards/staking account yet, you will receive the pool deposit fee on it!<br>

* **08a_genStakingAddrRetireCert.sh:** generates the de-registration certificate name.staking.dereg-cert to retire a stake-address form the blockchain
  <br>```./08a_genStakingAddrRetireCert.sh <name>```
  <br>```./08a_genStakingAddrRetireCert.sh owner``` generates the owner.staking.dereg-cert to retire the owner.staking.addr
  
* **08b_deregStakingAddrCert.sh:** re-register (retire) you stake-address with the **name.staking.dereg-cert certificate** with funds from name.payment.add from the blockchain.
  <br>```./08b_deregStakingAddrCert.sh <nameOfStakeAddr> <nameOfPaymentAddr>```
  <br>```./08b_deregStakingAddrCert.sh owner.staking owner.payment``` this will retire your owner staking address with the cert generated with script 08a from the blockchain.

* **10_genPolicy.sh:** generate policy keys, signing script and id as files **name.policy.skey/vkey/script/id**. You need a policy for Token minting.
  <br>```./10_genPolicy.sh <PolicyName>```
  <br>```./10_genPolicy.sh mypolicy``` this will generate the policyfiles with name mypolicy.policy.skey, mypolicy.policy.vkey, mypolicy.policy.script & mypolicy.policy.id

* **11a_mintAsset.sh:** mint/generate new Assets(Token) on a given payment address with a policyID generated before. This updates the Token Status File **policyname.tokenname.asset** for later usage when sending/burning Tokens.
  <br>```./11a_mintAsset.sh <AssetName> <AssetAmount to mint> <PolicyName> <nameOfPaymentAddr>```
  <br>```./11a_mintAsset.sh SUPERTOKEN 1000 mypolicy mywallet```<br>this will mint 1000 new SUPERTOKEN with policy 'mypolicy' on the payment address mywallet.addr
  <br>```./11a_mintAsset.sh MEGATOKEN 30 mypolicy owner.payment```<br>this will mint 30 new MEGATOKEN with policy 'mypolicy' on the payment address owner.payment.addr

* **11b_burnAsset.sh:** burnes Assets(Token) on a given payment address with a policyID you own the keys for. This updates the Token Status File **policyname.tokenname.asset** for later usage when sending/burning Tokens.
  <br>```./11b_burnAsset.sh <AssetName> <AssetAmount to mint> <PolicyName> <nameOfPaymentAddr>```
  <br>```./11b_burnAsset.sh SUPERTOKEN 22 mypolicy mywallet```<br>this will burn 22 SUPERTOKEN with policy 'mypolicy' on the payment address mywallet.addr
  <br>```./11b_burnAsset.sh MEGATOKEN 10 mypolicy owner.payment```<br>this will burn 10 MEGATOKEN with policy 'mypolicy' on the payment address owner.payment.addr

### poolname.pool.json

The json file could end up like this one after the pool was registered and also later de-registered.
```console
{
  "poolName": "mypool",
  "poolOwner": [
         {
         "ownerName": "owner",
         "ownerWitness": "local"
         }
         {
         "ownerName": "otherowner2",
         "ownerWitness": "external"
         }
   ],
  "poolRewards": "owner",
  "poolPledge": "100000000000",
  "poolCost": "500000000",
  "poolMargin": "0.10",
  "poolRelays": [
         {
         "relayType": "dns",
         "relayEntry": "relay-1.mypool.com",
         "relayPort": "3001"
         },
         {
         "relayType": "dns",
         "relayEntry": "relay-2.mypool.com",
         "relayPort": "3001"
         }
  ],
  "poolMetaName": "This is my Pool",
  "poolMetaDescription": "This is the description of my Pool!",
  "poolMetaTicker": "POOL",
  "poolMetaHomepage": "https://mypool.com",
  "poolMetaUrl": "https://mypool.com/mypool.metadata.json",
  "poolExtendedMetaUrl": "",
  "---": "--- DO NOT EDIT BELOW THIS LINE ---",
  "poolMetaHash": "f792c672a350971266b5404f04ff3bd47deb1544bc411566a2f95c090c1202cf",
  "regCertCreated": "So Mai 31 14:38:53 CEST 2020",
  "regCertFile": "mypool.pool.cert",
  "poolID": "68c2d7335f542f2d8b961bf6de5d5fd046b912b671868b30b79c3e2219f7e51a",
  "regEpoch": "21",
  "regSubmitted": "So Mai 31 14:39:46 CEST 2020",
  "deregCertCreated": "Di Jun  2 17:13:39 CEST 2020",
  "deregCertFile": "mypool.pool.dereg-cert",
  "deregEpoch": "37",
  "deregSubmitted": "Di Jun  2 17:14:38 CEST 2020"
}
```
</details>


### Filenames used and autolock for security

<details>
   <summary>Show all used naming schemes like *.addr, *.skey, *.pool.json, ... </summary>
   
I use the following naming scheme for the files:<br>
``` 
Simple "enterprise" address to only receive/send funds (no staking possible with these type of addresses):
name.addr, name.vkey, name.skey

Payment(Base)/Staking address combo:
name.payment.addr, name.payment.skey/vkey, name.deleg.cert
name.staking.addr, name.staking.skey/vkey, name.staking.cert/dereg-cert

Node/Pool files:
poolname.node.skey/vkey, poolname.node.counter, poolname.pool.cert/dereg-cert, poolname.pool.json, poolname.metadata.json
poolname.vrf.skey/vkey, poolname.pool.id, poolname.pool.id-bech
poolname.kes-xxx.skey/vkey, poolname.node-xxx.opcert (xxx increments with each KES generation = poolname.kes.counter)
poolname.kes.counter, poolname.kes-expire.json

ONLINE<->OFFLINE transfer files:
offlineTransfer.json

ITN witness files:
poolname.itn.skey/vkey
```

New for Hardware-Wallet (Ledger/Trezor) support:<br>
```
Hardware-SigningFile for simple "enterprise" address:
name.hwsfile (its like the .skey)

Hardware-SigningFile for Payment(Base)/Staking address combo:
name.payment.hwsfile, name.staking.hwsfile (its like the .skey)

Witness-Files for PoolRegistration or PoolReRegistration:
poolname_name_id.witness
```

New in Mary-Era:<br>
```
Policy files:
policyname.policy.skey/vkey, policyname.policy.script, policyname.policy.id

(Multi)Assets:
policyname.tokenname.asset
```

The *.addr files contains the address in the format "addr1vyjz4gde3aqw7e2vgg6ftdu687pcnpyzal8ax37cjukq5fg3ng25m" for example.
If you have an address and you wanna use it for later just do a simple:<br>
```echo "addr1vyjz4gde3aqw7e2vgg6ftdu687pcnpyzal8ax37cjukq5fg3ng25m" > myaddress.addr```

</details>

### File autolock

For a security reason, all important generated files are automatically locked against deleting/overwriting them by accident! Only the scripts will unlock/lock some of them automatically. If you wanna edit/delete a file by hand like editing the name.pool.json simply do a:<br>
```
chmod 600 poolname.pool.json
nano poolname.pool.json
chmod 400 poolname.pool.json
```

### Directory Structure

<details>
   <summary>Checkout how to use the scripts with directories... </summary>

There is no directory structure, the current design is FLAT. So all Examples below are generating/using files within the same directory. This should be fine for the most of you. If you're fine with this, skip this section and check the [Scriptfile Syntax](#scriptfiles-syntax) below.<p>However, if you wanna use directories there is a way: 
* **Method-1:** Making a directory for a complete set: (all wallet and poolfiles in one directory)
1. Put the scripts in a directory that is in your PATH environment variable, so you can call the scripts from everywhere.
1. Make a directory whereever you like
1. Call the scripts from within this directory, all files will be generated/used in this directory<p>
* **Method-2:** Using subdirectories from a base directory:
1. Put the scripts in a directory that is in your PATH environment variable, so you can call the scripts from everywhere.
1. Make a directory that is your BASE directory like /home/user/cardano
1. Go into this directory ```cd /home/user/cardano``` and make other subdirectories like ```mkdir mywallets``` and ```mkdir mypools```
1. **Call the scripts now only from this BASE directory** and give the names to the scripts **WITH** the directory in a relative way like (examples):
   <br>```03a_genStakingPaymentAddr.sh mywallets/allmyada``` this will generate your StakeAddressCombo with name allmyada in the mywallets subdirectory
   <br>```05b_genDelegationCert.sh mypools/superpool mywallets/allmyada``` this will generate the DelegationCertificate for your StakeAddress allmyada to your Pool named superpool.
   So, just use always the directory name infront to reference it on the commandline parameters. And keep in mind, you have to do it always from your choosen BASE directory. Because files like the poolname.pool.json are refering also to the subdirectories. And YES, you need a name like superpool or allmyada for it, don't call the scripts without them.<br>
   :bulb: Don't call the scripts with directories like ../xyz or /xyz/abc, it will not work at the moment. Call them from the choosen BASE directory without a leading . or .. Thx!

</details>

> :bulb: **The examples below are using the scripts in the same directory, so they are listed with a leading ./**<br>
**If you have the scripts copied to an other directory reachable via the PATH environment variable, than call the scripts WITHOUT the leading ./ !**

# Examples in Online-Mode

The examples in here are for using the scripts in Online-Mode. Please get yourself familiar on how to use each single script, a detailed Syntax about each script can be found [here](#scriptfiles-syntax).<br>
Working in [Offline-Mode](#examples-in-offline-mode) introduces another step before and ofter each example, so you should understand the Online-Mode first.

:bulb: Make sure your 00_common.sh is having the correct setup for your system!

## Generating a normal address, register a stake address, register a stake pool

Lets say we wanna make ourself a normal address to send/receive ada, we want this to be nicknamed mywallet.
Than we want to make ourself a pool owner stake address with the nickname owner, also we want to register a pool with the nickname mypool. The nickname is only to keep the files on the harddisc in order, nickname is not a ticker!

<details>
   <Summary>Show Example...<br></summary>

1. First, we need a running node. After that make your adjustments in the 00_common.sh script so the variables are pointing to the right files and source it (```source ./00_common.sh```)
1. Generate a simple address to receive some ADA ```./02_genPaymentAddrOnly.sh mywallet```
1. Transfer some ADA to that new address mywallet.addr
1. Check that you received it using ```./01_queryAddress.sh mywallet```
1. Generate the owner stake/payment combo with ```./03a_genStakingPaymentAddr.sh owner```
1. Send yourself over some funds to that new address owner.payment.addr to pay for the registration fees
<br>```./01_sendLovelaces.sh mywallet owner.payment 10000000```<br>
If you wanna send over all funds from your mywallet call the script like
<br>```./01_sendLovelaces.sh mywallet owner.payment ALL```
1. Check that you received it using ```./01_queryAddress.sh owner.payment```
1. Register the owner stakeaddress on the blockchain ```./03b_regStakingAddrCert.sh owner.staking owner.payment```
1. (Optional: you can verify that your stakeaddress in now on the blockchain by running<br>```./03c_checkStakingAddrOnChain.sh owner``` if you don't see it, wait a little and retry)
1. Generate the keys for your coreNode
   1. ```./04a_genNodeKeys.sh mypool```
   1. ```./04b_genVRFKeys.sh mypool```
   1. ```./04c_genKESKeys.sh mypool```
   1. ```./04d_genNodeOpCert.sh mypool```
1. Now you have all the key files to start your coreNode with them
1. Make sure you have enough funds on your owner.payment.addr to pay the pool registration fee in the next steps. Make sure to make your fund big enough to stay above the pledge that we will set in the next step.
1. Generate your stakepool certificate
   1. ```./05a_genStakepoolCert.sh mypool```<br>will generate a prefilled mypool.pool.json file for you, edit it
   1. We want 200k ADA pledge, 10k ADA costs per epoch and 8% pool margin so let us set these and the Metadata values in the json file like
   ```console
   {
      "poolName": "mypool",
      "poolOwner": [
         {
         "ownerName": "owner",
         "ownerWitness": "local"
         }
      ],
      "poolRewards": "owner",
      "poolPledge": "200000000000",
      "poolCost": "10000000000",
      "poolMargin": "0.08"
      "poolRelays": [
         {
         "relayType": "dns",
         "relayEntry": "relay.mypool.com",
         "relayPort": "3001"
         }
      ],
      "poolMetaName": "This is my Pool",
      "poolMetaDescription": "This is the description of my Pool!",
      "poolMetaTicker": "POOL",
      "poolMetaHomepage": "https://mypool.com",
      "poolMetaUrl": "https://mypool.com/mypool.metadata.json",
      "poolExtendedMetaUrl": "",
      "---": "--- DO NOT EDIT BELOW THIS LINE ---"
   }
   ```
   1. Run ```./05a_genStakepoolCert.sh mypool``` again with the saved json file, this will generate the mypool.pool.cert file
1. Delegate to your own pool as owner -> pledge ```./05b_genDelegationCert.sh mypool owner``` this will generate the owner.deleg.cert
1. :bulb: **Upload** the generated ```mypool.metadata.json``` file **onto your webserver** so that it is reachable via the URL you specified in the poolMetaUrl entry! Otherwise the next step will abort with an error.
1. Register your stakepool on the blockchain ```./05c_regStakepoolCert.sh mypool owner.payment```    

Done.
</details>

## Generating & register a stake address, just delegating to a stakepool

Lets say we wanna create a payment(base)/stake address combo with the nickname delegator and we wanna delegate the funds in the payment(base) address of that to the pool yourpool. (You'll need the yourpool.node.vkey for that.)

<details>
   <Summary>Show Example...<br></summary>

1. First, we need a running node. After that make your adjustments in the 00_common.sh script so the variables are pointing to the right files.
1. Generate the delegator stake/payment combo with ```./03a_genStakingPaymentAddr.sh delegator```
1. Send over some funds to that new address delegator.payment.addr to pay for the registration fees and to stake that also later
1. Register the delegator stakeaddress on the blockchain ```./03b_regStakingAddrCert.sh delegator.staking delegator.payment```<br>Other example: ```./03b_regStakingAddrCert.sh delegator.staking mywallet``` Here you would use the funds in mywallet to pay for the fees.
1. (Optional: you can verify that your stakeaddress in now on the blockchain by running<br>```./03c_checkStakingAddrOnChain.sh delegator``` if you don't see it instantly, wait a little and retry the same command)
1. Generate the delegation certificate delegator.deleg.cert with ```./05b_genDelegationCert.sh yourpool delegator```
1. Register the delegation certificate now on the blockchain with funds from delegator.payment.addr<br>```./06_regDelegationCert.sh delegator delegator.payment```

Done.
</details>

## Update stakepool parameters on the blockchain

If you wanna update you pledge, costs, owners or metadata on a registered stakepool just do the following

<details>
   <Summary>Show Example...<br></summary>

1. [Unlock](#file-autolock) the existing mypool.pool.json file and edit it. Only edit the values above the "--- DO NOT EDIT BELOW THIS LINE ---" line, save it again. 
1. Run ```./05a_genStakepoolCert.sh mypool``` to generate a new mypool.pool.cert file from it
1. :bulb: **Upload** the new ```mypool.metadata.json``` file **onto your webserver** so that it is reachable via the URL you specified in the poolMetaUrl entry! Otherwise the next step will abort with an error.
1. (Optional create delegation certificates if you have added an owner or an extra rewards account with script 05b)
1. Re-Register your stakepool on the blockchain with ```./05c_regStakepoolCert.sh mypool owner.payment```<br>No delegation update needed.

Done.  
</details>

## Claiming rewards on the Shelley blockchain

I'am sure you wanna claim some of your rewards that you earned running your stakepool. So lets say you have rewards in your owner.staking address and you wanna claim it to the owner.payment address.

<details>
   <Summary>Show Example...<br></summary>

1. You can always check that you have rewards in your stakeaccount by running ```./01_queryAddress.sh owner.staking```
1. Now you can claim your rewards by running ```./01_claimRewards.sh owner.staking owner.payment```
   This will claim the rewards from the owner.staking account and sends it to the owner.payment address, also owner.payment will pay for the transaction fees. It is only possible to claim all rewards, not only a part of it.<br>
   :bulb: ATTENTION, claiming rewards costs transaction fees! So you have two choices for that: The destination address pays for the transaction fees, or you specify an additional account that pays for the transaction fees. You can find examples for that above at the script 01_claimRewards.sh description.

Done.  

### Claiming rewards from the ITN Testnet with only SK/PK keys

If you ran a stakepool on the ITN and you only have your owner SK ed25519(e) and VK keys you can claim your rewards now<br>

1. Convert your ITN keys into a Shelley Staking Address by running: 
   <br>```./0x_convertITNtoStakeAddress.sh <StakeAddressName> <Private_ITN_Key_File>  <Public_ITN_Key_File>```
   <br>```./0x_convertITNtoStakeAddress.sh myitnrewards mypool.itn.skey mypool.itn.vkey```
   <br>This will generate a new Shelley stakeaddress with the 3 files myitnrewards.staking.skey, myitnrewards.staking.vkey and myitnrewards.staking.addr
1. You can check now your rewards by running ```./01_queryAddress.sh myitnrewards.staking```
1. You can claim your rewards by running ```./01_claimRewards.sh myitnrewards.staking destinationaccount``` like a normal rewards claim procedure, example above!

Done.  
</details>

## Register a multiowner stake pool

It's similar to a single owner stake pool registration (example above). All owners must have a registered stake address on the blockchain first! Here is a 2 owner example ...

<details>
   <summary>Show Example...</summary>

1. Generate the stakepool certificate
   1. ```./05a_genStakepoolCert.sh mypool```<br>will generate a prefilled mypool.pool.json file for you, edit it for multiowner usage and set your owners and also the rewards account. The rewards account is also a stake address (but not delegated to the pool!):
    ```console
   {
      "poolName": "mypool",
      "poolOwner": [
         {
         "ownerName": "owner-1",
         "ownerWitness": "local"
         },
         {
         "ownerName": "owner-2",
         "ownerWitness": "local"
         }
      ],
      "poolRewards": "rewards-account",
      "poolPledge": "200000000000",
      "poolCost": "10000000000",
      "poolMargin": "0.08"
   ...
   ```
   1. Run ```./05a_genStakepoolCert.sh mypool``` again with the saved json file, this will generate the mypool.pool.cert file
1. Delegate all owners to the pool -> pledge
<br>```./05b_genDelegationCert.sh mypool owner-1``` this will generate the owner-1.deleg.cert
<br>```./05b_genDelegationCert.sh mypool owner-2``` this will generate the owner-2.deleg.cert
1. Register your stakepool on the blockchain ```./05c_regStakepoolCert.sh mypool paymentaddress```    

Done.
</details>

## Using multiple relays in your poolname.pool.json

You can mix'n'match multiple relay entries in your poolname.pool.json file, below are a few common examples.

<details>
   <summary>Show Example...<br></summary>

### Using two dns named relay entries

Your poolRelays array section in the json file should look similar to:

```console
  "poolRelays": [
         {
         "relayType": "dns",
         "relayEntry": "relay-1.mypool.com",
         "relayPort": "3001"
         },
         {
         "relayType": "dns",
         "relayEntry": "relay-2.mypool.com",
         "relayPort": "3001"
         }
  ],
```

### Using a mixed relay setup

Your poolRelays array section in the json file should like similar to:

```console
  "poolRelays": [
         {
         "relayType": "dns",
         "relayEntry": "relay.mypool.com",
         "relayPort": "3001"
         },
         {
         "relayType": "ip",
         "relayEntry": "287.10.10.1",
         "relayPort": "3001"
         }
  ],
```

### Using three ipv4 named relay entries

Your poolRelays array section in the json file should like similar to:

```console
  "poolRelays": [
         {
         "relayType": "ip",
         "relayEntry": "287.10.10.1",
         "relayPort": "3001"
         },
         {
         "relayType": "ip",
         "relayEntry": "287.10.0.1",
         "relayPort": "3002"
         },
         {
         "relayType": "ip",
         "relayEntry": "317.10.0.1",
         "relayPort": "3001"
         }
  ],
```
</details>


## Retire a stakepool from the blockchain

If you wanna retire your registered stakepool mypool, you have to do just a few things

<details>
   <summary>Show Example...<br></summary>

1. Generate the retirement certificate for the stakepool mypool from data in mypool.pool.json<br>
   ```./07a_genStakepoolRetireCert.sh mypool``` this will retire the pool at the next epoch
1. De-Register your stakepool from the blockchain with ```./07b_deregStakepoolCert.sh mypool owner.payment```
 
Done.
</details>

## Retire a stakeaddress from the blockchain

If you wanna retire the staking address owner, you have to do just a few things

<details>
   <Summary>Show Example...<br></summary>

1. Generate the retirement certificate for the stake-address ```./08a_genStakingAddrRetireCert.sh owner```<br>this will generate the owner.staking.dereg-cert file
1. De-Register your stake-address from the blockchain with ```./08b_deregStakingAddrCert.sh owner.staking owner.payment```<br>you don't need to have funds on the owner.payment base address. you'll get the keyDepositFee back onto it!
1. You can check the current status of your onchain registration via the script 03c like<br>
   ```./03c_checkStakingAddrOnChain.sh owner```<br>If it doesn't go away directly, wait a little and retry this script.
 
Done.
</details>

## ITN-Witness Ticker check for wallets and Extended-Metadata.json Infos

<details>
   <summary>Explore how to use your ITN Ticker as Proof and also how to use extended-metadata.json</summary>
   
There is now an implementation of the extended-metadata.json for the pooldata. This can hold any kind of additional data for the registered pool. We see some Ticker spoofing getting more and more, so new people are trying to take over the Ticker from the people that ran a stakepool in the ITN and built up there reputation. There is no real way to forbid a double ticker registration, however, the "spoofing" stakepoolticker can be shown in the Daedalus/Yoroi/Pegasus wallet as a "spoof", so people can see this is not the real pool. I support this in my scripts. To anticipate in this (it is not fixed yet) you will need a "**jcli**" binary on your machine with the right path set in ```00_common.sh```. Prepare two files in the pool directory:
<br>```<poolname>.itn.skey``` this textfile should hold your ITN secret/private key
<br>```<poolname>.itn.vkey``` this textfile should hold your ITN public/verification key
<br>also you would need to add an additional URL **poolExtendedMetaUrl** for the next extended metadata json file on your webserver to your ```<poolname>.pool.json``` file like:
```console
   .
   .
   .
   "poolMetaHomepage": "https://mypool.com",
   "poolMetaUrl": "https://mypool.com/mypool.metadata.json",
   "poolExtendedMetaUrl": "https://mypool.com/mypool.extended-metadata.json",
   "---": "--- DO NOT EDIT BELOW THIS LINE ---"
  }
``` 
When you now generate your pool certificate, not only your ```<poolname>.metadata.json``` will be created as always, but also the ```<poolname>.extended-metadata.json``` that is holding your ITN witness to proof your Ticker ownership from the ITN. Upload BOTH to your webserver! :-)

Additional Feature: If you wanna also include the extended-metadata format Adapools is currently using you can do so by providing additional metadata information in the file ```<poolname>.additional-metadata.json``` !<br>
You can find an example of the Adapools format [here](https://a.adapools.org/extended-example).<br>
So if you hold a file ```<poolname>.additional-metadata.json``` with additional data in the same folder, script 05a will also integrate this information into the ```<poolname>.extended-metadata.json``` :-)<br>

</details>

## How to do a voting for SPOCRA in a simple process

<details>
   <summary>Explore how to vote for SPOCRA</summary>
   
We have created a simplified script to transmit a voting.json file on-chain. This version will currently be used to submit your vote on-chain for the SPOCRA voting.<br>A Step-by-Step Instruction on how to create the voting.json file can be found on Adam Dean's website -> [Step-by-Step Instruction](https://vote.crypto2099.io/SPOCRA-voting/).<br>
After you have generated your voting.json file you simply transmit it in a transaction on-chain with the script ```01_sendVoteMeta.sh``` like:<br> ```./01_sendVoteMeta.sh mywallet myvote```<br>This will for example transmit the myvote.json file (you name it without the .json) with funds from your wallet with the name mywallet.<br>
Thats it. :-)

</details>











# Examples in Offline-Mode

The examples in here are for using the scripts in Offine-Mode. Please get yourself familiar first with the scripts in [Online-Mode](#examples-in-online-mode). Also a detailed Syntax about each script can be found [here](#scriptfiles-syntax). Working offline is like working online, all is working in Offline-Mode, theses are just a few examples. :smiley:<br>

:bulb: Make sure your 00_common.sh is having the correct setup for your system!

**Understand the workflow in Offline-Mode:**

* **Step 1 : On the Online-Machine**
  Query up2date information about your address balances, rewards, blockchain-parameters...<br>
  If you wanna pay offline from your mywallet1.addr, just add the information for that.
  If you wanna claim rewards from your mywallet.staking address and you wanna pay with your smallwallet1.addr for that, just add these two addresses to the information. You need to add the information of your addresses you wanna pay with or you wanna claim rewards from, nothing more.<br>
  Update the **offlineTransfer.json file with ./01_workOffline.sh** and send(:floppy_disk:) it over to the Offline-Machine.

* **Step 2 : On the Offline-Machine**
  Do your normal work with the scripts like sending lovelaces or tokens from address to address, updating your stakepool parameters, claiming your rewards, etc...<br>
  Sign the transactions on the Offline-Machine, they will be automatically stored in the offlineTransfer.json. If you wanna do multiple transactions at the same time, use a few small payment wallets for this, because you can only pay from one individual wallet in an offline transaction at the same time. So if you wanna claim your rewards and also update your pool parameters, use two small payment wallets for that.<br>All offline transactions and also updated files like your pool.metadata.json or pool.extended-metadata.json will be stored in the offlineTransfer.json if you say so.<br>
  When you're finished, send(:floppy_disk:) the offlineTransfer.json back to your Online-Machine.

* **Step 3 : On the Online-Machine**
  **Execute the offline signed transactions** and/or extract files from the offlineTransfer.json like your updated pool.metadata.json file for example with **./01_workOffline.sh**<br>
  You're done, if you wanna continue to do some work: Gather again the latest balance informations from the address you wanna work with and send the offlineTransfer.json back to your Offline-Machine. And so on...<br>
  The offlineTransfer.json is your little carry bag for your balance/rewards information, transactions and files. :-)

**Config-Settings on the Online- / Offline-Machine:**

* Online-Machine: Set the ```offlineMode="no"``` parameter in the 00_common.sh, common.inc or ~/.common.inc config file.<br>Make sure you have a running and fully synced cardano-node on this Machine. Also cardano-cli.

* Offline-Machine: Set the ```offlineMode="yes"``` parameter in the 00_common.sh, common.inc or ~/.common.inc config file.<br>You only need the cardano-cli on this Machine, no cardano-node binaries.


## Generate some wallets for the daily operator work

So first you should create yourself a few small wallets for the daily Operator work, there is no need to use your big-owner-pledge-wallet for this every time. Lets say we wanna create three small wallets with the name smallwallet1, smallwallet2 and smallwallet3. And we wanna fund them via daedalus for example.

<details>
   <summary>Show Example...</summary>

<br>**Online-Machine:**

1. Make a fresh version of the offlineTransfer.json by running ```./01_workOffline.sh new```

:floppy_disk: Transfer the offlineTransfer.json to the Offline-Machine.

**Offline-Machine:**

1. Create three new payment-only wallets by running<br>```./02_genPaymentAddrOnly.sh smallwallet1```<br>```./02_genPaymentAddrOnly.sh smallwallet2```<br>```./02_genPaymentAddrOnly.sh smallwallet3```
1. Add the three new smallwallet1/2/3.addr files to your offlineTransfer.json<br>```./01_workOffline.sh attach smallwallet1.addr```<br>```./01_workOffline.sh attach smallwallet2.addr```<br>```./01_workOffline.sh attach smallwallet3.addr```

:floppy_disk: Transfer the offlineTransfer.json to the Online-Machine.

**Online-Machine:**

1. Extract the three included address files to the Online-Machine<br>```./01_workOffline.sh extract```

You have now successfully brought over the three files smallwallet1.addr, smallwallet2.addr and smallwallet3.addr to your Online-Machine. You can check the current balance on them like you did before running ```./01_queryAddress.sh smallwallet1```<br>
Ok, now fund those three small wallets via daedalus for example. Of course you can also do this from your big-owner-pledge-wallet offline via multiple steps, but we're just learning the steps together, so not overcomplicate the things. :-)<br>
You can of course use your already made and funded wallets for the following examples, we just need a starting point here.

</details>

## Generate a pool owner address, register a stake pool

We want to make a pool owner stake address the nickname owner, also we want to register a pool with the nickname mypool. The nickname is only to keep the files on the harddisc in order, nickname is not a ticker! We use the smallwallet1&2 to pay for the different fees in this process. Make sure you have enough funds on smallwallet1 & smallwallet2 for this registration.

<details>
   <Summary>Show Example...<br></summary>

<br>**Online-Machine:**

1. Add/Update the current UTXO balance for smallwallet1 in the offlineTransfer.json by running<br>```./01_workOffline.sh add smallwallet1``` (smallwallet1 will pay for the stake-address registration, 2 ADA + fees)
1. Add/Update the current UTXO balance for smallwallet2 in the offlineTransfer.json by running<br>```./01_workOffline.sh add smallwallet2``` (smallwallet2 will pay for the pool registration, 500 ADA + fees)

:floppy_disk: Transfer the offlineTransfer.json to the Offline-Machine.

**Offline-Machine:** (same steps like working online)

1. Generate the owner stake/payment combo with ```./03a_genStakingPaymentAddr.sh owner```
1. Attach the newly created payment and staking address into your offlineTransfer.json for later usage on the Online-Machine<br>```./01_workOffline.sh attach owner.payment.addr```<br>```./01_workOffline.sh attach owner.staking.addr```
1. Generate the owner stakeaddress registration transaction and pay the fees with smallwallet1<br>```./03b_regStakingAddrCert.sh owner.staking smallwallet1```
1. Generate the keys for your coreNode
   1. ```./04a_genNodeKeys.sh mypool```
   1. ```./04b_genVRFKeys.sh mypool```
   1. ```./04c_genKESKeys.sh mypool```
   1. ```./04d_genNodeOpCert.sh mypool```
1. Now you have all the key files to start your coreNode with them
1. Generate your stakepool certificate
   1. ```./05a_genStakepoolCert.sh mypool```<br>will generate a prefilled mypool.pool.json file for you, edit it
   1. We want 200k ADA pledge, 10k ADA costs per epoch and 4% pool margin so let us set these and the Metadata values in the json file like
   ```console
   {
      "poolName": "mypool",
      "poolOwner": [
         {
         "ownerName": "owner",
         "ownerWitness": "local"
         }
      ],
      "poolRewards": "owner",
      "poolPledge": "200000000000",
      "poolCost": "10000000000",
      "poolMargin": "0.04"
      "poolRelays": [
         {
         "relayType": "dns",
         "relayEntry": "relay.mypool.com",
         "relayPort": "3001"
         }
      ],
      "poolMetaName": "This is my Pool",
      "poolMetaDescription": "This is the description of my Pool!",
      "poolMetaTicker": "POOL",
      "poolMetaHomepage": "https://mypool.com",
      "poolMetaUrl": "https://mypool.com/mypool.metadata.json",
      "poolExtendedMetaUrl": "",
      "---": "--- DO NOT EDIT BELOW THIS LINE ---"
   }
   ```
   
   :bulb: You can find more details on the scripty-syntax [here](#scriptfiles-syntax)
   
1. Run ```./05a_genStakepoolCert.sh mypool``` again with the saved json file, this will generate the mypool.pool.cert file.<br>:bulb: If you wanna protect your TICKER a little more against others, contact me and you will get a unique TickerProtectionKey for your Ticker! If you already have one, run ```./05a_genStakepoolCert.sh <PoolNodeName> <your registration protection key>```<br>
1. Delegate to your own pool as owner -> pledge ```./05b_genDelegationCert.sh mypool owner``` this will generate the owner.deleg.cert
1. Generate the stakepool registration transaction and pay the fees with smallwallet2<br>```./05c_regStakepoolCert.sh mypool smallwallet2```<br>Let the script also autoinclude your new mypool.metadata.json file into the transferOffline.json    

:floppy_disk: Transfer the offlineTransfer.json to the Online-Machine.

**Online-Machine:**

1. Extract all the attached files (mypool.metadata.json, owner.payment.addr, owner.staking.addr) from the transferOffline.json<br>```./01_workOffline.sh extract```
1. Now would be the time to upload the mypool.metadata.json file to your webserver.
1. We submit the first cued transaction (stakekey registration) to the blockchain by running<br>```./01_workOffline.sh execute```
1. And now we submit the second cued transaction (stakepool registration) to the blockchain by running<br>```./01_workOffline.sh execute``` again

You can check the balance of your owner.payment and the rewards of owner.staking with the ```./01_queryAddress.sh``` script. Make sure to transfer enough ADA to your owner.payment account so you respect the registered pledge amount.

Done.
</details>


## Update stakepool parameters on the blockchain

Lets pretend you already have registered your stakepool 'mypool' in the past using theses scripts, now lets update some pool parameters like pledge, fees or the description for the stakepool(metadata). We use the smallwallet1 to pay for this update.

<details>
   <summary>Show Example...</summary>

<br>**Online-Machine:**

1. Add/Update the current UTXO balance for smallwallet1 in the offlineTransfer.json by running<br>```./01_workOffline.sh add smallwallet1```

:floppy_disk: Transfer the offlineTransfer.json to the Offline-Machine.

**Offline-Machine:** (same steps like working online)

1. [Unlock](#file-autolock) the existing mypool.pool.json file and edit it. Only edit the values above the "--- DO NOT EDIT BELOW THIS LINE ---" line, save it again. 
1. Run ```./05a_genStakepoolCert.sh mypool``` to generate a new mypool.pool.cert, mypool.metadata.json file from it
1. (Optional create delegation certificates if you have added an owner or an extra rewards account with script 05b)
1. Generate the offline Re-Registration of your stakepool with ```./05c_regStakepoolCert.sh mypool smallwallet1```<br>Your transaction with your updated pool-certificate is now stored in the offlineTransfer.json. As you have noticed, the 05c script also asked you if it should include the (maybe new) metadata files also in the offlineTransfer.json. So you need only one file for the transfer, we can extract them on the Online-Machine.

:floppy_disk: Transfer the offlineTransfer.json to the Online-Machine.

**Online-Machine:**
1. If your metadata/extended-metadata.json has changed and is in the transferOffline.json, extract it via<br>```./01_workOffline.sh extract```
1. Now would be the time to upload the new metadata/extended-metadata.json files to your webserver. If they have not changed at all, skip this step of course.
1. Finally we submit the created offline transaction now to the blockchain by running<br>```./01_workOffline.sh execute```

Done.  
</details>

## Claiming rewards on the Shelley blockchain

I'am sure you wanna claim some of your rewards that you earned running your stakepool. So lets say you have rewards in your owner.staking address and you wanna claim it to the owner.payment address by paying with funds from smallwallet2.

<details>
   <Summary>Show Example...<br></summary>

<br>**Online-Machine:**

Make sure you have your owner.staking.addr and smallwallet2.addr file on your Online-Machine, if not, copy it over from your Offline-Machine like a normal filecopy or use the attach->extract method we used in the example [here](#generate-some-wallets-for-the-daily-operator-work)

1. Add/Update the current UTXO balance for smallwallet2 in the offlineTransfer.json by running<br>```./01_workOffline.sh add smallwallet2```
1. Add/Update the current rewards state for owner.staking in the offlineTransfer.json by running<br>```./01_workOffline.sh add owner.staking```

Now we have the up2date information about the payment address smallwallet2 and also the current rewards state of owner.staking in the offlineTransfer.json.

:floppy_disk: Transfer the offlineTransfer.json to the Offline-Machine.

**Offline-Machine:** (same steps like working online)

1. You can claim your rewards by running ```./01_claimRewards.sh owner.staking owner.payment smallwallet2```
   This will claim the rewards from the owner.staking account and sends it to the owner.payment address, smallwallet2 will pay for the transaction fees.<br>
   :bulb: ATTENTION, claiming rewards costs transaction fees! So you have two choices for that: The destination address pays for the transaction fees, or you specify an additional account that pays for the transaction fees like we did now. You can find examples for that above at the script 01_claimRewards.sh description.

:floppy_disk: Transfer the offlineTransfer.json to the Online-Machine.

**Online-Machine:**
1. Execute the created offline rewards claim now on the blockchain by running<br>```./01_workOffline.sh execute```

Done.  

</details>

## Sending some funds from one address to another address

Lets say you wanna transfer 1000 Ada from your big-owner-payment-wallet owner.payment to a different address like smallwallet3 in this example.
Also you wanna transfer 20 ADA from smallwallet1 to smallwallet3 at the same time, only transfering the offlineTransfer.json once. 

<details>
   <Summary>Show Example...<br></summary>

<br>**Online-Machine:**

Make sure you have your owner.payment.addr and smallwallet1.addr file on your Online-Machine, if not, copy it over from your Offline-Machine like a normal filecopy or use the attach->extract method we used in the example [here](#generate-some-wallets-for-the-daily-operator-work)

1. Add/Update the current UTXO balance for owner.payment in the offlineTransfer.json by running<br>```./01_workOffline.sh add owner.payment```
1. Add/Update the current UTXO balance for smallwallet1 in the offlineTransfer.json by running<br>```./01_workOffline.sh add smallwallet1```

:floppy_disk: Transfer the offlineTransfer.json to the Offline-Machine.

**Offline-Machine:** (same steps like working online)

1. Generate the transaction to transfer 1000000000 lovelaces from owner.payment to smallwallet3<br>```./01_sendLovelaces.sh owner.payment smallwallet3 1000000000```
1. Generate the transaction to transfer 20000000 lovelaces from smallwallet1 also smallwallet3<br>```./01_sendLovelaces.sh smallwallet1 smallwallet3 20000000```

:floppy_disk: Transfer the offlineTransfer.json to the Online-Machine.

**Online-Machine:**

1. Execute the first created offline transaction now on the blockchain by running<br>```./01_workOffline.sh execute```
1. Execute the second created offline transaction now on the blockchain by running<br>```./01_workOffline.sh execute``` again

Done.  

</details>

# Conclusion

As you can see, its always the same procedure working in Offline-Mode:

1. Get the information about your payment/rewards addresses online using ./01_workOffline.sh
1. Transfer the offlineTransfer.json to the Offline-Machine
1. Do your normal operations on the Offline-Machine (only one payment from an individual payment address)
1. Transfer the offlineTransfer.json to the Online-Machine
1. Execute the operation online on the chain, and/or extract some included files too using ./01_workOffline.sh

If you have questions, feel free to contact me via telegram: @atada_stakepool

Best regards,
 Martin

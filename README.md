# Node Setup on Loyal




![ew](https://user-images.githubusercontent.com/108979536/200149733-d4275bb5-3e25-4340-b9b2-94a5f5d62fd2.png)


What is Loyal?
Loyal is a universal rewards program running on a sovereign layer-1 blockchain powered by Ignite and Cosmos-SDK with the goal of onboarding shoppers to Web3 and giving back with LoyalCares


# Hardware Requirements

### Minimum Hardware Requirements

  + 4x CPUs; the faster clock speed the better

  + 8GB RAM

  + 100GB of storage (SSD or NVME)

 ### Recommended Hardware Requirements
 
  + 8x CPUs; the faster clock speed the better

  + 64GB RAM

  + 1TB of storage (SSD or NVME)

# 1-ɪɴᴛᴀʟʟᴀᴛɪᴏɴ ᴏɴᴇ ᴛɪᴍᴇ (ꜱᴄʀɪᴘᴛ ᴀᴜᴛᴏᴍᴀᴛɪᴄ ɪɴꜱᴛᴀʟʟᴀᴛɪᴏɴ)


      wget -O loyal.sh https://raw.githubusercontent.com/appieasahbie/loyal/main/loyal.sh && chmod +x loyal.sh && ./loyal.sh
      
      
      
# Post installation

     source $HOME/.bash_profile    
     
     
#  Open ports

     sudo ufw default allow outgoing
     sudo ufw default deny incoming
     sudo ufw allow ssh/tcp
     sudo ufw limit ssh/tcp
     sudo ufw allow ${LOYAL_PORT}656,${LOYAL_PORT}660/tcp
     sudo ufw enable

# Create wallet

     loyald keys add $WALLET
     
  ###   To recover your old wallet use this command

     loyald keys add $WALLET --recover
     
  ### Show current keys
  
     loyald keys list
     
  ### Add wallet and valoper address and load variables into the system
 
     LOYAL_PORT=13
     echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
     if [ ! $WALLET ]; then
	       echo "export WALLET=wallet" >> $HOME/.bash_profile
     fi
     echo "export LOYAL_CHAIN_ID=loyal-1" >> $HOME/.bash_profile
     echo "export LOYAL_PORT=${LOYAL_PORT}" >> $HOME/.bash_profile
     source $HOME/.bash_profile
     
     
 ### Fund your wallet

     curl -X POST -d '{"address": "'"Your wallet address"'", "coins": ["10000000ulyl"]}' "https://faucet.joinloyal.io/"
     
     
 ### Check your balance
 
     loyald query bank balances $LOYAL_WALLET_ADDRESS
     
     
 ### Create validator
 
      loyald tx staking create-validator \
      --amount 1000000ulyl \
      --from $WALLET \
      --commission-max-change-rate "0.1" \
      --commission-max-rate "0.3" \
      --commission-rate "0.08" \
      --min-self-delegation "1" \
      --pubkey  $(loyald tendermint show-validator) \
      --moniker $NODENAME \
      --chain-id $LOYAL_CHAIN_ID --gas=200000 --fees 500ulyl
      
      
#  Service management
 
  ### Check your logs

      journalctl -fu loyald -o cat
      
      
  ### Start service
  
      sudo systemctl start loyald
      
  ### Stop service
  
      sudo systemctl stop loyald
      
### Restart service

      sudo systemctl restart loyald
      
### Synchronization info

     loyald status 2>&1 | jq .SyncInfo
 
### Validator info

     loyald status 2>&1 | jq .ValidatorInfo
     
### Node info

     loyald status 2>&1 | jq .NodeInfo
     
### Show node id

     loyald tendermint show-node-id    
     
     
###  Validator list

     loyald q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl

### Delegate stake

      loyald tx staking delegate $LOYAL_VALOPER_ADDRESS 1000000ulyl --from=$WALLET --chain-id=$LOYAL_CHAIN_ID --gas=200000 --fees 500ulyl -y
      
###  Redelegate stake from validator to another validator

       loyald tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 1000000ulyl --from=$WALLET --chain-id=$LOYAL_CHAIN_ID --gas=200000 --fees 250ulyl -y
       
### Unjail

         loyald tx slashing unjail --from $WALLET --chain-id $LOYAL_CHAIN_ID --gas=200000 --fees 500ulyl
         
### Delete your node

        sudo systemctl stop loyald
        sudo systemctl disable loyald
        sudo rm /etc/systemd/system/loyald* -rf
        sudo rm $(which loyald) -rf
        sudo rm $HOME/.loyal* -rf
        sudo rm $HOME/loyal -rf
        sed -i '/LOYAL_/d' ~/.bash_profile
      
      
      

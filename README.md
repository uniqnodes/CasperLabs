![alt text](https://github.com/CasperLabs/casper-node/raw/master/images/CasperLabs_Logo_Horizontal_RGB.png)

# casper-node (ubuntu 18.04)
### Pre-Requisites for Building  
`sudo apt-get update`  

install cmake  
`sudo apt-get -y install cmake`  

install rust  
`curl --proto '=https' --tlsv1.2 https://sh.rustup.rs/ -sSf | sh` (enter)  
`source $HOME/.cargo/env`  
   
install libssl-dev  
`sudo apt-get install -y libssl-dev`  
   
install pkg-config  
`sudo apt-get install -y pkg-config`  
   
install gcc, g++ and make  
`sudo apt install build-essential`  

install git  
`sudo apt install -y git`  

install jq  
`sudo apt-get install -y jq`  
   
### Clone repo  
`git clone https://github.com/CasperLabs/casper-node.git`  
`cd casper-node`  
`git fetch`  
`git checkout release-0.9.3`  
   
### Setup
`make setup-rs && make build-client-contracts -j`  
`curl -JLO https://bintray.com/casperlabs/debian/download_file?file_path=casper-node-launcher_0.3.1-0_amd64.deb`  
`curl -JLO https://bintray.com/casperlabs/debian/download_file?file_path=casper-client_0.9.3-0_amd64.deb`  
`sudo apt install -y ./casper-client_0.9.3-0_amd64.deb ./casper-node-launcher_0.3.1-0_amd64.deb`  
`cd /etc/casper`  
`sudo -u casper ./pull_casper_node_version.sh 1_0_1`  
Get trusted hash for config.toml  
```
sudo sed -i "/trusted_hash =/c\trusted_hash = '$(curl -s 18.144.176.168:8888/status | jq -r .last_added_block_info.hash | tr -d '\n')'" /etc/casper/1_0_1/config.toml
```  
`sudo logrotate -f /etc/logrotate.d/casper-node`  
`sudo /etc/casper/delete_local_db.sh; sleep 1`  

### Create keypair  
`sudo chmod -R 777 /etc/casper/validator_keys`  
`cd /etc/casper/validator_keys`  
`casper-client keygen`  if you created the key from Clarity, you can replace the files where in this folder with them. Or you can import the key created here to the account in Clarity.  
`sudo systemctl start casper-node-launcher`  
`systemctl status casper-node-launcher` (If you see the result "Active: active (running)", you can continue.)  

### Node Control
`cd ~`  
`cd casper-node`  
`wget https://raw.githubusercontent.com/matsuro-hadouken/casper-tools/master/explorer.sh`  
`chmod +x explorer.sh`  
`./explorer.sh` (ERA and HEIGH values the HOST field should be synchronized with the system, it may take around 1.5 hours.)  

### Bonding  
`cd ~`  
`cd casper-node`  

(before bonding, get faucet on Clarity)  
`casper-client put-deploy --chain-name delta-11 --node-address http://127.0.0.1:7777 --secret-key /etc/casper/validator_keys/secret_key.pem --session-path  $HOME/casper-node/target/wasm32-unknown-unknown/release/add_bid.wasm  --payment-amount 1000000000  --session-arg="public_key:public_key='<PUBLIC_KEY>'" --session-arg="amount:u512='900000000000'" --session-arg="delegation_rate:u8='10'"`  

`casper-client get-deploy <DEPLOY_HASH> | jq .result.execution_results`  

### Shut down node and clear state  
`sudo systemctl stop casper-node`  
`sudo /etc/casper/delete_local_db.sh`  

### Update from existing setup  
*clean up previous version nodes*  
`cd casper-node`  
`sudo systemctl stop casper-node-launcher.service`  
`sudo rm -rf /etc/casper/1_0_1`  
`sudo rm -rf /var/lib/casper/bin/1_0_1`  
`sudo rm -rf /var/lib/casper/casper-node`  
`sudo rm /etc/casper/casper-node-launcher-state.toml`  
`sudo apt remove -y casper-client casper-node-launcher`  
`curl -JLO https://bintray.com/casperlabs/debian/download_file?file_path=casper-client_0.9.3-0_amd64.deb`  
`curl -JLO https://bintray.com/casperlabs/debian/download_file?file_path=casper-node-launcher_0.3.1-0_amd64.deb`  
`sudo apt install -y ./casper-client_0.9.3-0_amd64.deb ./casper-node-launcher_0.3.1-0_amd64.deb`  
`sudo -u casper /etc/casper/pull_casper_node_version.sh 1_0_1 delta-11`  
`sudo -u casper /etc/casper/config_from_example.sh 1_0_1`  
```
sudo sed -i "/trusted_hash =/c\trusted_hash = '$(curl -s 18.144.176.168:8888/status | jq -r .last_added_block_info.hash | tr -d '\n')'" /etc/casper/1_0_1/config.toml
```  
`sudo logrotate -f /etc/logrotate.d/casper-node`  
`sudo systemctl start casper-node-launcher; sleep 2`  
`systemctl status casper-node-launcher`  

### Recompile  
`cd casper-node`  
`git fetch`  
`git checkout release-0.9.3`  
`make setup-rs && make build-client-contracts -j`

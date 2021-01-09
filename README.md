![alt text](https://github.com/CasperLabs/casper-node/raw/master/images/CasperLabs_Logo_Horizontal_RGB.png)

# casper-node (ubuntu 18.04)
### Pre-Requisites for Building  
`sudo apt-get update`  

install cmake  
`sudo apt-get -y install cmake`  

install rust  
`curl https://sh.rustup.rs -sSf | sh` (enter)  
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
`git checkout release-0.5.1`  
   
### Setup
`make setup-rs && make build-system-contracts -j && make build-client-contracts -j`  
`cargo build -p casper-node --release`  
`cd ~`  
`curl -JLO https://bintray.com/casperlabs/debian/download_file?file_path=casper-client_0.5.1-3583_amd64.deb`  
`curl -JLO https://bintray.com/casperlabs/debian/download_file?file_path=casper-node_0.5.1-3583_amd64.deb`  
`sudo apt install -y ./casper-node_0.5.1-3583_amd64.deb ./casper-client_0.5.1-3583_amd64.deb`  
Get trusted hash for config.toml  
`curl -s 18.144.176.168:8888/status | jq -r .last_added_block_info.hash`  
Then copy the result, open config.toml and paste it near the trusted_hash in single quotes and uncomment that line.  
`sudo nano /etc/casper/config.toml`  

### Create keypair  
`sudo chmod -R 777 /etc/casper/validator_keys`  
`cd /etc/casper/validator_keys`  
`casper-client keygen`  if you created the key from Clarity, you can replace the files where in this folder with them. Or you can import the key created here to the account in Clarity.  
`sudo systemctl start casper-node`  
`systemctl status casper-node` (If you see the result "Active: active (running)", you can continue.)  

### Bonding  
`cd ~`  
`cd casper-node`  
`wget https://raw.githubusercontent.com/matsuro-hadouken/casper-tools/master/active_validators.sh`  
`chmod +x active_validators.sh`  
`./active_validators.sh` (See what BID_AMOUNT value other members of the network are using and choose a value for yourself. This value will be used when bonding.)  
You must have gotten faucet on Clarity to bonding!  
`wget https://raw.githubusercontent.com/matsuro-hadouken/casper-tools/master/bond.sh`  
`sudo nano bond.sh` (Replace the PUB_KEY_HEX with your key and replace the BID_AMOUNT fields with your choosen.)  
`chmod +x bond.sh`  
`./bond.sh` (If you see a Transaction hash, the bonding is success.)  

`casper-client --help` (see for yourself what you can do ☺)  

### Shut down node and clear state  
`sudo systemctl stop casper-node`  
`sudo /etc/casper/delete_local_db.sh`

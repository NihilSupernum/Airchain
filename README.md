## How to Run a Validator Node on Airchains

### Prerequisites

Before you begin, make sure you have the following:

1. **A Linux-based server** (Ubuntu 20.04 or later is recommended).
2. **Basic knowledge of the command line** and server management.
3. **A registered account** with Airchains and some initial tokens to stake.

### Step 1: Set Up the Environment

1. **Update your system** to ensure all dependencies are up to date:

   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```

2. **Install necessary dependencies** like `git`, `curl`, and `build-essential`:

   ```bash
   sudo apt-get install git curl build-essential -y
   ```

### Step 2: Install Go

Airchains is built with Go, so you need to install it on your server.

1. **Download and install Go**:

   ```bash
   wget https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz
   sudo tar -C /usr/local -xzf go1.20.5.linux-amd64.tar.gz
   ```

2. **Add Go to your PATH**:

   ```bash
   echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
   source ~/.bashrc
   ```

3. **Verify the installation**:

   ```bash
   go version
   ```

### Step 3: Clone the Airchains Repository

1. **Clone the Airchains repository** to your server:

   ```bash
   git clone https://github.com/airchains/airchains-node
   cd airchains-node
   ```

2. **Checkout the latest stable release**:

   ```bash
   git checkout main
   ```

### Step 4: Build the Validator Node

1. **Compile the node software**:

   ```bash
   make build
   ```

   This will create the necessary binaries to run the node.

2. **Install the binaries**:

   ```bash
   sudo make install
   ```

### Step 5: Configure the Node

1. **Initialize the node**:

   ```bash
   airchaind init <your-node-name> --chain-id airchains-testnet
   ```

2. **Configure the node** to connect to the network by editing the `config.toml` file:

   ```bash
   nano ~/.airchains/config/config.toml
   ```

   Ensure you set the following parameters:
   
   - **Persistent peers**: Add a list of peer nodes to connect to.
   - **Moniker**: Your node’s name.
   - **Chain ID**: Set to `airchains-testnet`.

3. **Set up pruning options** to reduce disk usage:

   ```bash
   pruning = "custom"
   pruning-keep-recent = "100"
   pruning-interval = "10"
   ```

### Step 6: Start the Node

1. **Start the node** using systemd for stability:

   ```bash
   sudo tee /etc/systemd/system/airchaind.service > /dev/null <<EOF
   [Unit]
   Description=Airchains Validator Node
   After=network-online.target

   [Service]
   User=$USER
   ExecStart=$(which airchaind) start
   Restart=always
   RestartSec=3
   LimitNOFILE=4096

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

2. **Enable and start the service**:

   ```bash
   sudo systemctl enable airchaind
   sudo systemctl start airchaind
   ```

3. **Check the status** of your node:

   ```bash
   sudo systemctl status airchaind
   ```

### Step 7: Create a Validator

1. **Create a new wallet** or import an existing one:

   ```bash
   airchaind keys add <wallet-name>
   ```

2. **Fund your wallet** with enough tokens to stake as a validator.

3. **Create the validator**:

   ```bash
   airchaind tx staking create-validator \
   --amount <amount-of-tokens> \
   --pubkey $(airchaind tendermint show-validator) \
   --moniker <your-validator-name> \
   --chain-id airchains-testnet \
   --commission-rate "0.10" \
   --commission-max-rate "0.20" \
   --commission-max-change-rate "0.01" \
   --min-self-delegation "1" \
   --from <wallet-name> \
   --gas auto \
   --fees <fees> \
   --yes
   ```

4. **Verify your validator** status:

   ```bash
   airchaind query staking validator $(airchaind keys show <wallet-name> --bech val -a)
   ```

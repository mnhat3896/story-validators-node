## How to Use the Script
1. Save the script: Copy the bash script (script at the end of the post) into a `.sh` file (e.g., `install_story_node.sh`), or download the script in this repo

2. Make the script executable:
```
chmod +x install_story_node.sh
```

3. Run the script:
```
./install_story_node.sh
```

4. Follow prompts: When running, it will prompt you to input your moniker name.


## What the Script Does
- Installs dependencies: Ensures required packages like curl, git, jq, etc., are installed.
- Downloads binaries: Fetches and installs the story-geth and story binaries.
- Sets up system services: Creates and enables systemd services for story-geth and story.
- Initializes the Story node: Uses the inputted moniker to initialize the Story node.
- Creates a validator: Extracts the private key from the node configuration and creates a validator with the specified stake.

## Purpose
This script automates the installation and configuration of the Story node, allowing you to run and maintain a node seamlessly. It provides a streamlined process from installation to creating a validator, all in one run.


## Here is the script with one-liner

```
#!/bin/bash
read -p "Enter your moniker name: " moniker_name

sudo apt update && sudo apt-get update && sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 -y && wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.3-b224fdf.tar.gz && tar -xzvf geth-linux-amd64-0.9.3-b224fdf.tar.gz && [ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin && if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile; fi && sudo cp geth-linux-amd64-0.9.3-b224fdf/geth $HOME/go/bin/story-geth && source $HOME/.bash_profile && story-geth version && wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.10.1-57567e5.tar.gz && tar -xzvf story-linux-amd64-0.10.1-57567e5.tar.gz && [ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin && if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile; fi && cp $HOME/story-linux-amd64-0.10.1-57567e5/story $HOME/go/bin && source $HOME/.bash_profile && story version && story init --network iliad --moniker "$moniker_name" && sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF [Unit] Description=Story Geth Client After=network.target [Service] User=root ExecStart=/root/go/bin/story-geth --iliad --syncmode full Restart=on-failure RestartSec=3 LimitNOFILE=4096 [Install] WantedBy=multi-user.target EOF && sudo tee /etc/systemd/system/story.service > /dev/null <<EOF [Unit] Description=Story Consensus Client After=network.target [Service] User=root ExecStart=/root/go/bin/story run Restart=on-failure RestartSec=3 LimitNOFILE=4096 [Install] WantedBy=multi-user.target EOF && sudo systemctl daemon-reload && sudo systemctl start story-geth && sudo systemctl enable story-geth && sudo systemctl status story-geth && sudo systemctl daemon-reload && sudo systemctl start story && sudo systemctl enable story && sudo systemctl status story && priv_key=$(jq -r '.priv_key.value' /root/.story/story/config/priv_validator_key.json) && story validator create --stake 1000000000000000000 --private-key "$priv_key"
```

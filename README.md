# CAT Token Box Setup Guide

![CAT Token Box Banner](banner.jpeg)

VIEW UP TO DATE GUIDE HERE: [CAT TOKEN INSTALLATION GUIDE]([https://ubuntu.com/download/desktop](https://ordinalos.github.io/cat-token-box-install/))

> **HIGHLY RECOMMENDED:** To use the official documentation https://github.com/CATProtocol/cat-token-box

> **NOTE:** I will be updating this guide. Follow me at [@ordinalos](https://x.com/ordinalos) for updates.

## 1. System Requirements and Prerequisites

### 1.1 Operating System

This guide primarily focuses on a fresh Ubuntu installation. We recommend using the latest LTS version of Ubuntu, which you can download from the [official Ubuntu website](https://ubuntu.com/download/desktop).

### 1.2 VPS Option

If you prefer using a VPS, here are three recommended providers that offer easy Ubuntu setup:

- [DigitalOcean](https://www.digitalocean.com/)
- [Linode](https://www.linode.com/)
- [Vultr](https://www.vultr.com/)

### 1.3 Basic System Setup

Update your system and install basic packages:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y build-essential git curl
```

### 1.4 Install Node.js and Yarn

```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# Download and install Node.js (you may need to restart the terminal)
nvm install 20

# Verify the Node.js version
node -v # should print `v20.17.0`

# Verify the npm version
npm -v # should print `10.8.2`

# Install Yarn
npm install -g yarn
```

### 1.5 Install and Setup PostgreSQL

```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo -u postgres psql -c "CREATE DATABASE cat;"
sudo -u postgres psql -c "CREATE USER cat_user WITH PASSWORD 'your_password';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE cat TO cat_user;"
```

> **NOTE:** Replace 'your_password' with a strong, unique password.

#### Troubleshooting Section 1:

- To check if PostgreSQL is running: `sudo systemctl status postgresql`
- To test PostgreSQL connection: `psql -h localhost -U cat_user -d cat`

## 2. CAT Token Box Setup

### 2.1 Clone the Repository

```bash
git clone https://github.com/CATProtocol/cat-token-box.git
cd cat-token-box
```

### 2.2 Install Dependencies and Build

```bash
yarn install && yarn build
```

#### Troubleshooting Section 2:

- If build fails, ensure all prerequisites are installed: `node -v && yarn -v`
- Check for any error messages in the build output and address them accordingly.

## 3. Tracker Setup

For detailed instructions, refer to the [official tracker documentation](https://github.com/CATProtocol/cat-token-box/blob/main/packages/tracker).

### 3.1 Installation and Build

```bash
cd packages/tracker
yarn install
yarn build
```

### 3.2 Configure Environment

```bash
cp .env.example .env
nano .env
```

Update the .env file with your configuration. Here's an example of what your .env file might look like:

```
DATABASE_TYPE=postgres
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_DB=cat
DATABASE_USERNAME=cat_user
DATABASE_PASSWORD=your_password

RPC_HOST=127.0.0.1
RPC_PORT=8332
RPC_USER=bitcoin
RPC_PASSWORD=your_rpc_password

NETWORK=mainnet
API_PORT=3000
GENESIS_BLOCK_HEIGHT=0
```

Make sure to replace `your_password` and `your_rpc_password` with your actual PostgreSQL and RPC passwords.

### 3.3 Set Up Docker

```bash
sudo chmod 777 docker/data
sudo chmod 777 docker/pgdata
docker compose up -d
```

### 3.4 Run the Tracker Service

Option 1: Using Docker (Recommended)

```bash
cd ../../
docker build -t tracker:latest .
docker run -d \
    --name tracker \
    --add-host="host.docker.internal:host-gateway" \
    -e DATABASE_HOST="host.docker.internal" \
    -e RPC_HOST="host.docker.internal" \
    -p 3000:3000 \
    tracker:latest
docker logs -f tracker
```

Option 2: Using Yarn

```bash
# Development mode
yarn run start

# Production mode
yarn run start:prod
```

> **NOTE:** Ensure the tracker syncs to the latest block before running CLI commands. Check the sync progress at http://127.0.0.1:3000/api

#### Troubleshooting Section 3:

- To check if the tracker is running: `docker ps | grep tracker`
- To check tracker logs: `docker logs tracker`
- To check the API endpoint: `curl http://localhost:3000/api`
- To check sync progress, use the CLI to check wallet balance (see CLI commands below)

## 4. CLI Setup and Usage

For detailed instructions, refer to the [official CLI documentation](https://github.com/CATProtocol/cat-token-box/tree/main/packages/cli).

### 4.1 Installation and Build

```bash
cd packages/cli
yarn install
yarn build
```

### 4.2 Configuration

```bash
cp config.example.json config.json
nano config.json
```

Update config.json with your own configuration. Here's an example of what your config.json might look like:

```json
{
  "network": "testnet",
  "mnemonic": "your twelve word mnemonic phrase goes here",
  "trackerUrl": "http://localhost:3000"
}
```

Replace "your twelve word mnemonic phrase goes here" with your actual mnemonic phrase.

### 4.3 CLI Commands

```bash
# Create a wallet
yarn cli wallet create

# Show address
yarn cli wallet address

# Show token balances
yarn cli wallet balances

# Deploy token
yarn cli deploy --metadata=example.json
# or
yarn cli deploy --name=cat --symbol=CAT --decimals=2 --max=21000000 --premine=0 --limit=5

# Mint token
yarn cli mint -i [tokenId] [amount?]

# Send token
yarn cli send -i [tokenId] [receiver] [amount]
```

> **NOTE:** Replace [tokenId], [receiver], and [amount] with actual values when using these commands.

When deploying a token with a metadata file, here's an example of what your `example.json` might look like:

```json
{
    "name": "cat",
    "symbol": "CAT",
    "decimals": 2,
    "max": "21000000",
    "limit": "5",
    "premine": "0"
}
```

#### Troubleshooting Section 4:

- If CLI commands fail, ensure the tracker is fully synced by checking the wallet balance.
- To check tracker sync progress: `yarn cli wallet balance`
- If commands still fail, check tracker logs for any error messages: `docker logs tracker`

## 5. Final Notes

- Always keep your system and packages up to date.
- Regularly backup your database and wallet.
- Monitor your system resources, especially when running a full node.
- For advanced configurations or issues, refer to the official documentation or seek community support.

> **REMINDER:** Always refer to the official documentation and README files in each package for the most up-to-date information and troubleshooting tips.

I will be updating this guide. Follow me at [@ordinalos](https://x.com/ordinalos) for updates.

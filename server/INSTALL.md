# ao server install instructions


These instructions will assume you are setting up dctrl/ao on a fresh install of
- Ubuntu 17.10 - https://www.ubuntu.com/download/desktop
### 1. Get ubuntu

To create a bootable Ubuntu usb (on linux) download the .iso file from the above link. Run the command `sudo fdisk -l` to see a list of the the plugged in drives. Once you have selected the correct drive (ie. /dev/sdb) run: The dd command specifies an input file (if) which is the iso and an output file (of) which is the usd drive. Careful this formats the usb!
- `sudo unmount <path-to-usb>`
- `sudo dd if=<path-to-ubuntu-iso> of=<path-to-usb> bs=1M`
- i.e. `sudo dd if=./Downloads/Ubuntu-17.10.iso> of=/dev/sdb bs=1M`


This should create a bootable usb and you can then go through the ubuntu install process by interupting the computers boot up with `delete` or `f10`.

---
### 2. Get the database

Now that we have an operating system, the first thing we need is a database to store and persist our applications state. Our database of choice is rethinkdb. We are following the instructions from https://www.rethinkdb.com/docs/build/ with some modifications for 17.10 -->
- first download and unpack the source;
  - `cd Downloads`
  - `wget https://download.rethinkdb.com/dist/rethinkdb-2.3.6.tgz`
  - `tar xf rethinkdb-2.3.6.tgz`
- Now there should be a rethinkdb folder:
  - `cd rethinkdb-2.3.6`
- Now we can build, these commands take some time
  - ```sudo apt install build-essential protobuf-compiler python libprotobuf-dev libcurl4-openssl-dev libboost-all-dev libncurses5-dev libjemalloc-dev wget m4 clang libssl-dev```
  - `./configure --allow-fetch CXX=clang++`
  - `make`
  - `sudo make install`

After doing the above commands you should have rethinkdb installed. Use the command `which rethinkdb`. If installed correctly the path to the executable file (in my case /usr/local/bin/rethinkdb) will be printed. Now that we have rethinkdb installed we want to set it up so it runs automatically on boot. To do this we are setting up a systemd service.

---
### 3. Setup the database service
We need to create a directory for the rethinkdb data. I suggest following the convention of .bitcoind and .lnd :
- `rethinkdb create -d .rethinkdb`

We also need to create a config file,
- https://github.com/rethinkdb/rethinkdb/blob/next/packaging/assets/config/default.conf.sample
- `touch .rethinkdb/rethinkdb.conf`

Now that rethink is ready we can create a systemd service file `touch /etc/systemd/system/rethinkdb.service`. Edit the file with this replacing the user name with yours.
```text
[Unit]
Description=rethinkdb-deamon

[Service]
User=<user-name>
ExecStart=/usr/local/bin/rethinkdb serve --config-file /home/<user-name>/.rethinkdb/rethinkdb.conf --directory /home/<user-name>/.rethinkdb
Restart=always
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
Enable
  - `systemctl daemon-reload`
  - `systemctl enable rethinkdb.service`
  - `systemctl start rethinkdb.service`
  - Just useful to know:
  - `systemctl status rethinkdb.service` # see status
  - `journalctl -u rethinkdb.service` # see logs

To confirm check the admin console at `localhost:8080` is still on after a reboot.

---
### 3. Setup the bitcoin node
First get the bitcoin binaries from https://bitcoin.org/en/download and unpack them

- `cd Downloads`
- `tar xf bitcoin-0.16.0-x86_64-linux-gnu.tar.gz`

Now you can setup another service file `touch /etc/systemd/system/bitcoind.service`

```text
[Unit]
Description=bitcoin-deamon

[Service]
User=<user-name>
ExecStart=/home/<user-name>/Downloads/bitcoin-0.16.0/bin/bitcoind
Restart=always

[Install]
WantedBy=multi-user.target
```

We need a configuration file as well `touch /etc/systemd/system/bitcoind.service`.

```text
testnet=0
server=1

rpcuser=bitcoinrpc
rpcpassword=yourGoodSecretb7p5y4EpBsdfDSvcarw7udgXsAce
```

Now enable
- `systemctl daemon-reload`
- `systemctl enable bitcoind.service`
- `systemctl start bitcoind.service`

You can check the status of bitcoind using:
- `bitcoin-cli getnetworkinfo`
- `bitcoin-cli getblockchaininfo`
- `bitcoin-cli help`

Confirm that these command survive a reboot. It will take some time to sync the node.

---
# Setup node.js
You can install node

---
# Get the ao code

- `git clone `
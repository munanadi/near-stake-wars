# Near stake wars

Near protocol announced their [chunk only producers](https://near.org/papers/nightshade/#nightshade) as an improvement to their protocol that would implement the power of sharding to scale the TPS of the network. To encourage more participation and to further decentralize the network they came up with [Stake wars](https://near.org/blog/join-stake-wars-to-become-a-chunk-only-producer/)

I quikly hoped on the chance becuase who doesn't want to be a validator? (Okay, almost a validator). This is exciting stuff. Further to make this deal juicy they would *delegate tokens to your node* if you manage to keep doing the other stuff like monitoring and keeping it upto date.

---
#### Hardware Requirements

- `CPU`	4-Core CPU with AVX support
- `RAM`	8GB DDR4
- `Storage`	500GB SSD

You can choose to run the node in cloud or like myself you can run your bare metal server in your garage or something. I personally wanted to be close to the server and tinker around with it and chose to go bare metal. 

> My Config was [Ryzen 4650G](https://www.amd.com/en/products/apu/amd-ryzen-5-pro-4650g), 8GB of RAM, 500GB of NVMe SSD, [Asus A320 Motherboard](https://www.asus.com/Motherboards-Components/Motherboards/PRIME/PRIME-A320M-K/), 450W PSU. A Ethernet cable for a reliable connection.

> 💡 The more scraps you are able to find by hunting for deals or asking your friends around for parts, the more 💲 you save. You best friend is google here. Don't know if a CPU would support AVX? Google it. 

---
#### Hardware Setup

### 1. Need to get Ubunutu server installed. Here is a [video](https://www.youtube.com/watch?v=xUH256WAWt0)

### 2. Install and setup `ssh`
 This step is important because you can disconnect your monitor from your server and still connect to the server via your PC/laptop.
 
  1. `sudo apt install openssh-server` This will isntall the ssh server on your node. Later we will use a ssh client to connect to this.
  2. `sudo systemctl enable ssh` Enable the service that controls the ssh server
  3. `sudo systemctl start ssh` Start the ssh server
  4. `sudo systemctl status ssh` Check status of the ssh server
  
 > 💡 Here is a [guide](https://www.cyberciti.biz/faq/how-to-install-ssh-on-ubuntu-linux-using-apt-get/) . Stuck somewhere? Read the error on screen and google it! 
 
### 3. Get to know your local ip. 
With this we will be able to connect to the node over our local network connection

The command is `ip a`. Your output should look something like the below show. Look for `UP` and the address next to `inet` is your local ip

> Here in my case it is `192.168.1.2`. Watch out if your node is shutdown or restarted this might change as IP's are assigned dynamically

![image](https://user-images.githubusercontent.com/22261173/183836806-7ed04668-ed18-4e1c-aee8-794680ec385b.png)

### 4. Connecting from your PC

For this step you need to know what user id you entered while creating the server.

> My case it was `spoiler` and I named the node `spoilernear`

I use ubuntu and ssh comes out of the box. If you're using some other OS, A google (search)[https://www.google.com/search?q=use+ssh&oq=use+ssh&aqs=chrome..69i57j0i512l9.1097j0j9&sourceid=chrome&ie=UTF-8] will help you find it. 

Log on to your node using 
```
ssh <USER_ID>@<LOCAL_IP_ADDR>
```

> My values here were `USER_ID` = spoiler. `LOCAL_IP_ADDR` = 192.168.1.2 (This we got from the previous step)

You will be prompted to enter your password. Enter it and done! You're now logged into your node from your PC. 
Look at the image below for reference, See how the terminal changed to 
```
spoiler@spoilernear:~$ 
```
This shows I am logged in as `spoiler` in my `spoilernear` node.

> `spoilernear` is the name of my node I chose during the ubuntu server install.

![image](https://user-images.githubusercontent.com/22261173/183838761-0d1ef844-468d-4c4a-bb72-19cd90d2a657.png)

---

#### Creating a shardnet wallet

Swtich back to your PC for this and head to this [https://wallet.shardnet.near.org/](https://wallet.shardnet.near.org/)
Follow the instructions to create a shardnet wallet. This (video will help you out)[https://www.youtube.com/watch?v=SnMyoEm73mQ] though it creates a mainnet wallet, the steps are same for a shardnet wallet too.

---
### Setting up the node

Follow the `ssh` connection step above and switch back to your node now.

##### 1. Run these commands to install various tools
```
$ sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo

$ sudo apt install python3
$ sudo apt install docker-ce

$ sudo apt install python3-pip

$ sudo apt install clang build-essential make
```

##### 2. Set env using the below command
```
$ USER_BASE_BIN=$(python3 -m site --user-base)/bin
$ export PATH="$USER_BASE_BIN:$PATH"
```

##### 3. Install rust

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Proceed with the defualt installation.

After done installing, enter `source $HOME/.cargo/env` 

---
##### 4. Finally we download and setup the node.

###### 1. Clone the git project of nearcore from (here)[https://github.com/near/nearcore] by doing 
```
$ git clone https://github.com/near/nearcore
$ cd nearcore
$ git fetch
```

###### 2. Checkout the recommended commit from (here)[https://github.com/near/stakewars-iii/blob/main/commit.md]

```
$ git checkout <COMMIT> // COMMIT given in the above file
```

###### 3. Compile the nearcore binary so that you can run the node

```
$ cargo build -p neard --release --features shardnet
```

Now you should be able to find something like this below. Use `cd` to navigate around. 

![image](https://user-images.githubusercontent.com/22261173/183846681-48a8a825-0ac6-4b17-85d2-0f25a685abf7.png)

`neard` is the binary that can be executed that will run your node.

###### 4. Get configuration files

```
$ ./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
$ rm ~/.near/config.json
$ wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

This will fetch your configuration files that help your node with genesis configuration and other stuff. 

###### 5. Run your node so that it catches up with the latest block

```
$ cd ~/nearcore
$ ./target/release/neard --home ~/.near run
```

Downloading blocks and headers should look something like this.

![Screenshot from 2022-07-20 10-03-27](https://user-images.githubusercontent.com/22261173/183847937-6c4168b0-24ec-4e2f-92b3-2994c0052717.png)

Once it 100% done, you can `Ctrl + C` to exit out of this and stop the node for now.

###### 6. Setting up your account and wallet on the node.

Setup your `near-cli` on your node. Log in with your shardnet account. 

> Remember to set NEAR_ENV=shardnet or use that in front of your commands to execute them on shardnet like `NEAR_ENV=shardnet near login`

```
$ near login
```

Do this and follow the steps. Look for image below for reference. 

> Note the `wallet.shardnet..` in the url that is provided. Anything other than `shardnet` wont work. Check your `NEAR_ENV=shardnet` again.

![image](https://user-images.githubusercontent.com/22261173/183849501-97c3c7d7-b222-41ec-8ebb-9ed51151b1fd.png)

###### 7. Generate a key for your account

```
near generate-key <POOL_ID>
```

`POOL_ID` will be `wallet_name.factory.shardnet.near` 

> Here I chose my wallet name to be `shardoor` => `POOL_ID` for me was `shardoor.factory.shardnet.near`

![image](https://user-images.githubusercontent.com/22261173/183851317-c25e3c1c-f94c-43ba-8fcc-db102466579f.png)

###### 8. Make sure all keys are in place

- You will need a `validator_key.json` in `~/.near`. This makes sure that the node can actually sign on the chunks it produces.

If this doesn't exists create the file and copy the keys from the file that is producede in the previous step. 

> Remember to rename `account_id` from `WALLET.shardnet.near` to `WALLET.factory.shardnet.near`. My `WALLET` here was `shardoor`

> Also rename `private_key` to `secret_key`

- `WALLET.shardnet.near.json` in `~/.near-credentials/shardnet/...` is where your wallet key resides. This is also used for signing.

Your file then should look something like shown below.

![image](https://user-images.githubusercontent.com/22261173/183853329-7385e210-749d-47fb-b77c-9d359e6ab59c.png)

###### That's it. Your node is all setup for producing those chunks.

---

#### Run the validator in the background.

This will let you monitor your node and explore around doing other stuff while it runs in the background

```
$ sudo vi /etc/systemd/system/neard.service
```

This opens a `vi` editor. Just enter `i` to enter into `insert` mode and paste the following stuff

```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=<USER>
#Group=near
WorkingDirectory=/home/<USER>/.near
ExecStart=/home/<USER>/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

> <USER> here is your appropiate location. In my case it is `spoiler`. ie. You should be able to `cd` to the paths as shown below in the picture

![image](https://user-images.githubusercontent.com/22261173/183855560-acef4aa4-7085-4709-bc27-6d0449388638.png)

Enable the service and start the service

```
$ sudo systemctl enable neard
$ sudo systemctl start neard
```

To check if the service is running properly,

```
$ sudo systemctl status neard
```

Something like this will show up. If it says `active` then it means just that

![image](https://user-images.githubusercontent.com/22261173/183856321-a7954fad-3ee1-4a60-88af-502881ceeba8.png)

If for any reason you needed to change the config or want to restart the node, the below command should do the job.

```
$ sudo systemctl restart neard
```

##### To check if your node is running, we can check the logs using

```
$ journalctl -n 100 -f -u neard
```

To make this colorful 

```
$ sudo apt install ccze
$ journalctl -n 100 -f -u neard | ccze -A
```


---
### Setting up the stake pools and pings

###### 1. Mounting a staking pool

Get your stuff from here
- `POOL_NAME` is what you generated the key with in the above step. I chose it as `shardoor`
- `ACCOUNT_ID` is my wallet name. Which is `shardoor`
- `PUBLIC_KEY` We can get this from doing `$ cat ~/.near-credentials/shardnet/shardoor.shardnet.near.json` and copy the public key form here.

```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<POOL_NAME>", "owner_id": "<ACCOUNT_ID>", "stake_public_key": "<PUBLIC_KEY>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<ACCOUNT_ID>" --amount=30 --gas=300000000000000
```

You can check if this call was successful by chekcing this (https://explorer.shardnet.near.org/accounts/<ACCOUNT_ID.shardet.near>)[https://explorer.shardnet.near.org/accounts/shardoor.shardnet.near]. Go throught the transcations and see if the call to create the staking pool was successful or not.

###### 2. Stake and Deposti NEAR

Get your stuff
- `POOL_ID` is `POOL_NAME.factory.shardnet.near`. Mine is `shardoor.factory.shardnet.near`.
- `AMOUNT` is how much you want to stake. You want to stake `500 NEAR` Enter 500. The seat price was around 700 when I started the competition.
- `ACCOUNT_ID` This is your wallet name again.

```
near call <POOL_ID> deposit_and_stake --amount <AMOUNT> --accountId <ACCOUNT_ID> --gas=300000000000000
```

> Again the explorer can be used to debug if this actually worked.

###### 3. Need to keep pinging every epoch to singal to others that you're still in fact alive and doing your job.

- `POOL_ID` is `POOL_NAME.factory.shardnet.near`. Mine is `shardoor.factory.shardnet.near`.
- `ACCOUNT_ID` This is your wallet name again.

```
$ near call <POOL_ID> ping '{}' --accountId <ACCOUNT_ID> --gas=300000000000000
```

> Check the explorer link to see if your pings are working. If for some reason this doesn't work then you need to check your keys in `~/.near-credentials/shardnet/WALLET.shradenet.near.json`. The combination of the keys might be wrong could be the reason this is not able to verify the transactions.

---

#### General pointers

1. You need to `ping` from your node in every epoch. This can be done with a cron jon.

2. It takes a few epochs for the node to start producing chunks. 

---

#### Price of my setup

I found the baremetal cost to be manageable so I went shopping around and bought the hardware. It really depends on how much stuff you can put together and your local market prices. 

I spend around 24K INR. (~ 300 USD) for my setup.

> Electricity cost can also vary from city to city. So you'll need to factoy in for that as well.


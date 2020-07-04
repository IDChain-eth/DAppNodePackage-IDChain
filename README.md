# BrightID IDChain DAppNode package

General information about IDChain is available at https://github.com/BrightID/IDChain.

## Installation
The latest package version is available on [DAppNode explorer](https://dappnode.github.io/explorer/#/repo/0x6a111e20889ace99ca14c1ab38cf6c1176ed0ae7). To directly install the latest version follow this link: 

**http://my.dappnode/#/installer/idchain.public.dappnode.eth**

If you just want to run a node, there are no extra actions necessary. Install the DAppNodePackage as usual.
Once the node is running you will see the sync status on the Dashboard.

### Connecting to your IDChain instance
The rpc API of your personal IDChain node is available via:
 - http: `http://idchain.dappnode:8545`
 - Websocket: `ws://idchain.dappnode:8545`

## Become IDChain validator
> Note: `$EXTRA_OPTS` and `$SYNCMODE` variables can be modified on your DAppNode at http://my.dappnode/#/packages/idchain.public.dappnode.eth/config.

### Preparation
 - Change `$EXTRA_OPTS` to include `--rpcapi personal,clique`
 - Attach to your IDChain instance on the dappnode: `geth attach http://idchain.dappnode:8545`
 - Create or import an account to be used for validating, using geth `personal` namespace functions 
 (see https://geth.ethereum.org/docs/rpc/ns-personal)
 - If you created a new account, make sure to backup the keystore. Go to 
 http://my.dappnode/#/packages/idchain.public.dappnode.eth/file-manager and enter `/idchain/keystore` 
 in the _DOWNLOAD FILE FROM PACKAGE_ form. Download keystore.zip and store in a secure location.
 - Inform other validators to propose your account for validating
 - Check clique status to see if you are approved using `clique.status()`

### Start validating
 Once you are approved you have to restart idchain with mining enabled and your validating account unlocked:
 - Create a txtfile with the passphrase required to unlock the validator account: `echo <PASSPHRASE> > passphrase.txt`
 - Upload `passphrase.txt` to the dappnode filesystem into `/idchain/` folder (Use http://my.dappnode/#/packages/idchain.public.dappnode.eth/file-manager)
 - Change `$EXTRA_OPTS` to unlock your signer account and start mining. It should include `--miner.gasprice 10000000000 
 --mine --unlock <your signer account address> --password /idchain/passphrase.txt --allow-insecure-unlock`. (`--allow-insecure-unlock`
 is a tolerable risk, as the rpc api is only exposed over the DappNode VPN. See [Security section](#Security) below for alternative setup)
 - Change `$SYNCMODE` to `full`
 - Save new config. Your IDChain node will restart and should start validating blocks!

### Security
To further lockdown the IDChain instance you can do the following:
- Disable rpc access by removing `--rpcapi <modules>` and `-allow-insecure-unlock` from `$EXTRA_OPTS`

Now you can only attach to your IDChain instance via SSH:
1. SSH into your dappnode host
1. Execute ```docker exec -ti DAppNodePackage-idchain.public.dappnode.eth /usr/local/bin/geth attach /idchain/geth.ipc```

The downside of this change is that you no longer can connect to your IDChain instance with Metamask.

### Restore validating account
To import your validator account from the saved keystore.zip:
- extract `keystore.zip`
- open the package filemanager at http://my.dappnode/#/packages/idchain.public.dappnode.eth/file-manager
- Click `Browse` and select the keystore file extracted before
- Set the upload path to `/idchain/keystore/`
- Click `Uploac`

# DAppNode package Development

### Preconditions
 - Install dappnode SDK: `npm install -g @dappnode/dappnodesdk`

### Build and Publish
1. commit changes to `dev` branch. (to publish a new release of idchain just update the `git clone` line in `build/DockerFile`)
1. If the IDChain version changes: Update `upstreamVersion` in `dappnode_package.json` to match the IDChain version
1. Build the DAppNode package (`dappnodesdk build`) and test installation on your DAppNode 
1. If package works as expected, merge changes to `master` branch (via pull request from `dev` to `master`)
1. Checkout master branch
1. Increase version number: `dappnodesdk increase <major|minor|patch>
1. Build the DAppNode package: `dappnodesdk build`
1. Prepare publishing: `npx @dappnode/dappnodesdk publish <major|minor|patch>`
1. Publish through BDEV agent following [instructions below](#publish-through-aragon-bdev-agent)
1. Commit all changed files to the master branch.

### Publish through Aragon BDEV agent
**Preconditions**
 - "frame.sh" is installed
 - "hot" (your BDev token holding) account and "Smart" (The BDev agent) account are imported in frame
 - Metamask extension is disabled in your browser
 - Frame extension is enabled in your browser

#### Create vote to Publish package:
1. Open the pre-filled publish link (The last line of `dappnodeSDK publish` command)
1. Double-check that the developer address is set to BDEV agent (0x7099b4d99876480fcc0ac46d7809b7287a946050)
1. Double-check that your "hot" account is selected in Frame
1. Click "Connect" Button. This will actually connect the page with web3 from the Frame extension.
   1. If this is the first time you use frame with dappnode, approve the connection request from `my.dappnode` in frame
1. Click "Publish" button
1. Approve the transaction in Frame

Now a new vote should be created in BDEV Aragon! (Similar to e.g. https://mainnet.aragon.org/#/brightiddev/0x52fcd2e55bb7eb78ff83b6a5648d452b5671cec4/vote/58/)

Once the vote gets enacted, the BDEV agent will execute the publish transaction.

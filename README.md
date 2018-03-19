# shares-contract

https://hackernoon.com/ethereum-smart-contracts-from-zero-to-end-to-end-dapp-ebae43cdf859

Ethereum Smart Contracts for Beginners: From Zero to End-to-End DApp

Before we begin

Be sure to have node ≥ 8 installed on your machine. This will make writing our async/await functionality easier in the future.

If you would prefer to follow along with the completed code: https://github.com/qimingfang/shares-contract
What is a Smart Contract?

If you’re have an object oriented programming background, think of it like this:

    Smart Contracts == classes
    When you “deploy” a smart contract to the public blockchain, you get an address for the Smart Contract (in the blockchain), analogous to how the return value of new Object() is an address (in memory)
    We can write code to interact with the specific instances of “live” Smart Contracts, by calling methods on them. Read methods are quick; write methods take a while because we need to wait for the next block to be mined.
    Contracts can follow standards (like OO interfaces), such as the ERC-20 standard. Contracts that follow this standard must implement required methods.

Set up Truffle

Truffle is a framework for developing Ethereum Smart Contracts. It provides a nice file structure layout, testing, console, debugging, and a handful of tooling.

npm install -g truffle

Starting a new truffle project is simple and straightforward.

$ mkdir shares
$ cd shares
$ truffle init

You’ll notice that truffle sets up 3 directories for you:

    contracts — this is where your smart contracts will go. You’ll notice that there is a Migrations.sol file already. This is needed in order to run migrations (to deploy your smart contract) later.
    migrations — this is where you will write your migrations. Migrations are used to deploy your Smart Contracts.
    test — mocha style tests for the contracts

There is also a truffle.config file. This is used to configure environments (which RPC node to use). Yes, you can use Truffle to deploy your Smart Contracts to mainnet and testnet.
Write a Smart Contract in Solidity

Here is a bare bone hello world style Smart Contract. It is shamelessly taken from the uport-demo repo.
contracts/shares.sol

This contract is a wrapper around a mapping (address => uint) object. You can think of a mapping in Solidity as a Map<Address, Int> in Java, and NSDictionary in Swift.

This contract exposes 2 public methods: a getter and a setter. Simple enough, right?
Testing the Smart Contract

Not only is it slow to deploy your contracts to the mainnet / testnet for testing purposes, but it’s also immutable and may cost you a considerate amount of money. It is therefore very important to adopt TDD when developing Smart Contracts.

We’ll use chai to test the contact, since chai gives us a lot of nice syntax for asserting that things are equal. Instead of importing assert directly from node.js, and asserting equality with the junit styleassert.equal(a, b), we can instead leverage the more readable hamcrest style a.should.be.equal(b).

First, let’s install some dependencies

$ npm init
$ npm install --save-dev --save-exact chai chai-as-promised chai-bignumber

Then we create 2 tests

    to test that the contract was created correctly
    to test that updateShares and getShares work as expected

test/shares.test.js

Run truffle test to make sure that you see the tests pass.
Deploy to local blockchain

The smart contract is written and tested. We are confident of its business logic and correctness. Now it’s time to deploy this onto a local blockchain for testing.

First, let’s set up a local blockchain with ganache (download and install it). Ganache comes pre-poulated with 10 test accounts (with 100 ETH in each). Also note the RPC server URL. We’ll need it shortly.
Ganache

Next, we’ll need to create a migration
migrations/2_deploy_contracts.js

Finally, we need to add ganache into our truffle configurations. The port should correspond to the port number from ganache.
truffle.js

Ready to deploy the contract to your ganache local blockchain?

$ truffle migrate --network development

You should start seeing the contract showing up under the Transactions tab.
Ganache showing contract deployment
Deploy onto Rinkeby Test Net

Contracts are more useful when you can deploy them onto public blockchains. To connect to a test net, we need to run another RPC client that points at an Ethereum Test Net (we will use Rinkeby for this, but there are a few more test nets available).

First, you’ll need geth.

Next, you’ll need to use geth to create some Rinkeby accounts

geth --rinkeby account new

List your Rinkeby geth accounts to make sure that you were able to create an account.

geth --rinkeby account list

Once that’s done, you’ll need some test ETH. You can follow the instructions on this faucet and claim some test Rinkeby ETH.

Once the transaction is confirmed, you should be able to see it on the rinkeby etherscan UI.

https://rinkeby.etherscan.io/address/<your address here>

To deploy contracts onto Rinkeby, you’ll need to start geth and point it at the rinkeby network. You’ll need to unlock the wallet you just created (with ETH from the faucet) so that truffle can interact with it.

./build/bin/geth --rinkeby --rpc --rpcapi db,eth,net,web3,personal --unlock="<your address here>"

For example, it would look like this for me

./build/bin/geth --rinkeby --rpc --rpcapi db,eth,net,web3,personal --unlock="0xcfd31d9ffb5bc1b1f4b7af99cd4792f2b11da185"

You’ll need to wait a bit for geth to sync blocks on the network. While that’s syncing, let’s set up another truffle environment for Rinkeby.

Be sure to replace the from field to your account’s public key. The choice of gas is a bit arbitrary. Just choose a sufficiently high number that doesn’t exceed the block gas limit.
truffle.js

Here is how you can find out about what the block gas limit is

Your geth is probably still syncing. In the meantime we can take a look at truffle console and get an introduction to web3.

web3 is a js library (typically embedded client side into HTML) used to interface with smart contracts.

$ truffle console --network rinkeby
$ truffle(rinkeby)> truffle(rinkeby)> web3.eth.getBlock('latest')

Here, you will see the latest block that your geth client has reported in. It has a lot of the contents we’ve heard about, such as gasLimit, nonce (for mining), parentHash (to form a chain), etc. Neat stuff.

You can find more docs on web3 here.
Wait for geth to sync

… buffering …
Geth finishes Syncing — Deploy! 🎉

Alright, time to run the migration on Rinkeby!

$ truffle migrate --network rinkeby

As the contracts are deploying, you’ll notice that under the hood, it’s actually generating contract creation transactions on the Rinkeby testnet.

When you ran the migrations, you should have seen a contract address in terminal that points to the instance of our shares contract. You can also view this contract on etherscan.
Web3.js — the frontend

Now that our contract is deployed, we can now interact with it using a simple web DApp:

    include the web3.js script in <head>
    instantiate the web3 instance with a local geth
    instantiate the contract with an abi file (this is the interface file that comes when you compile your contracts. You can find this by looking at the compiled json in /build/contracts/shares.json
    find the specific deployed contract with the interface in (3)
    call functions the contract interface accordingly

Every time you click on the buy button, the url to the transaction will be posted to the console. You can click on the URL to watch your transaction complete. Once it completes, you can refresh the page, and see the number of shares increase.

Completed code here: https://github.com/qimingfang/shares-contract
Conclusion / Learnings

In my opinion, despite the wide adoption, there are a lot of tooling that is missing in the solidity development space. Some (technical and non-technical) things I noticed:

    Debugging was difficult — truffle debug did not really work for me.
    there was no easy UI of testing the contract on a local testnet, other than through truffle console. This is akin to testing node.js apps by going into node instead of using Postman or other HTTP clients.
    There is no easy way to guarantee that your contracts are correct. You can write tests and test every code path, but that seldom happens in the development world. Testing web UI with the contract interface is even trickier. The standard right now is to use audited contracts as bases. Open-Zeppelin is very simple to set up.
    In addition to solidity, there are a variety of other frameworks (and languages) to choose from. This increases the risk even further; not only do you have to write correct smart contracts, the frameworks have to compile these correctly to the Ethereum bytecode.
    Contracts are immutable. I’ve gotten various opinions from experts in how to upgrade these contracts. Proxying seems to be the standard way of doing this now, and it requires the same amount of audit/rigor as developing the contract itself. Imagine a full code audit every time a rest-api server gets deployed. That’d be nuts!
    A lot of work is done at the protocol levels right now. I don’t think that’s a coincidence. There is currently massive network effects going on for protocols, since other protocols will be built on top of base protocols. One of the cool things about working in Blockchain is that no one knows about the correct protocol based incentive model (inflation, participation, etc) that will lead to massive network effects. Of course, if you end up building a killer app for a killer use case, you’d probably go back and define a protocol for the killer app.

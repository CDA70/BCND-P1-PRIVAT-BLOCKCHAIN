# Private Blockchain Application

You are starting your journey as a Blockchain Developer, this project allows you to demonstrate
that you are familiarized with the fundamentals concepts of a Blockchain platform.
Concepts like: - Block - Blockchain - Wallet - Blockchain Identity - Proof of Existance

Are some of the most important components in the Blockchain Framework that you will need to describe and also
why not? Implement too.

In this project you will have a boilerplate code with a REST Api already setup to expose some of the functionalities
you will implement in your private blockchain.

## What problem will you solve implementing this private Blockchain application?

Your employer is trying to make a test of concept on how a Blockchain application can be implemented in his company.
He is an astronomy fans and he spend most of his free time on searching stars in the sky, that's why he would like
to create a test application that will allows him to register stars, and also some others of his friends can register stars
too but making sure the application know who owned each star.

### What is the process describe by the employer to be implemented in the application?

1. The application will create a Genesis Block when we run the application.
2. The user will request the application to send a message to be signed using a Wallet and in this way verify the ownership over the wallet address. The message format will be: `<WALLET_ADRESS>:${new Date().getTime().toString().slice(0,-3)}:starRegistry`;
3. Once the user have the message the user can use a Wallet to sign the message.
4. The user will try to submit the Star object for that it will submit: `wallet address`, `message`, `signature` and the `star` object with the star information.
   The Start information will be formed in this format:
   ```json
       "star": {
           "dec": "68° 52' 56.9",
           "ra": "16h 29m 1.0s",
           "story": "Testing the story 4"
   	}
   ```
5. The application will verify if the time elapsed from the request ownership (the time is contained in the message) and the time when you submit the star is less than 5 minutes.
6. If everything is okay the star information will be stored in the block and added to the `chain`
7. The application will allow us to retrieve the Star objects belong to an owner (wallet address).

## What tools or technologies you will use to create this application?

- This application will be created using Node.js and Javascript programming language. The architecture will use ES6 classes
  because it will help us to organize the code and facilitate the maintnance of the code.
- The company suggest to use Visual Studio Code as an IDE to write your code because it will help you debug the code easily
  but you can choose the code editor you feel confortable with.
- Some of the libraries or npm modules you will use are:
  - "bitcoinjs-lib": "^4.0.3",
  - "bitcoinjs-message": "^2.0.0",
  - "body-parser": "^1.18.3",
  - "crypto-js": "^3.1.9-1",
  - "express": "^4.16.4",
  - "hex2ascii": "0.0.3",
  - "morgan": "^1.9.1"
    Remember if you need install any other library you will use `npm install <npm_module_name>`

Libraries purpose:

1. `bitcoinjs-lib` and `bitcoinjs-message`. Those libraries will help us to verify the wallet address ownership, we are going to use it to verify the signature.
2. `express` The REST Api created for the purpose of this project it is being created using Express.js framework.
3. `body-parser` this library will be used as middleware module for Express and will help us to read the json data submitted in a POST request.
4. `crypto-js` This module contain some of the most important cryotographic methods and will help us to create the block hash.
5. `hex2ascii` This library will help us to **decode** the data saved in the body of a Block.

## Understanding the boilerplate code

The Boilerplate code is a simple architecture for a Blockchain application, it includes a REST APIs application to expose the your Blockchain application methods to your client applications or users.

1. `app.js` file. It contains the configuration and initialization of the REST Api, the team who provide this boilerplate code suggest do not change this code because it is already tested and works as expected.
2. `BlockchainController.js` file. It contains the routes of the REST Api. Those are the methods that expose the urls you will need to call when make a request to the application.
3. `src` folder. In here we are going to have the main two classes we needed to create our Blockchain application, we are going to create a `block.js` file and a `blockchain.js` file that will contain the `Block` and `BlockChain` classes.

### Starting with the boilerplate code:

First thing first, we are going to download or clone our boilerplate code.

Then we need to install all the libraries and module dependencies, to do that: open a terminal and run the command `npm install`

**( Remember to be able to work on this project you will need to have installed in your computer Node.js and npm )**

At this point we are ready to run our project for first time, use the command: `node app.js`

You can check in your terminal the the Express application is listening in the PORT 8000

## What do I need to implement to satisfy my employer requirements?

# block.js implementation

## the validate() function validates if the block has been tampered or not.

- Return a new promise to allow the method be called asynchronous.
  \
   `return new Promise((resolve, reject) => {}`

- Create an auxiliary variable and store the current hash of the block in it (this represent the block object)
  \
   `let currentBlockHash = self.hash;`

- Recalculate the hash of the entire block (Use SHA256 from crypto-js library)
  \
   `self.hash = SHA256(JSON.stringify(self)).toString();`

- Compare if the auxiliary hash value is different from the calculated one.
  \
   `if (currentBlockHash != self.hash) {`
- Resolve true or false depending if it is valid or not.

```
if (currentBlockHash != self.hash) {
                // Returning the Block is not valid
                resolve("The block is not valid!");
            }
            else {
                // Returning the Block is valid
                resolve(currentBlockHash);

            }
```

## Modify the 'getBData()' function to return the block body (decoding the data)

- Use hex2ascii module to decode the data
  \
  `let encodeData = this.body;`
  \
  `let decodedData = hex2ascii(encodeData);`
- Because data is a javascript object use JSON.parse(string) to get the Javascript Object
  \
  `let object = JSON.parse(decodedData);`

Resolve with the data and make sure that you don't need to return the data for the genesis block OR reject with an error.

```
if (this.height >= 0) {
                resolve(object);
            } else {
                reject(object);
            }
```

# Blockchain.js implementation

## Modify the '\_addBlock(block)' function to store a block in the chain

the function must return a Promise that will resolve with the block added OR reject if an error happen during the execution.
height must be checked to assign the previousBlockHash.
\

- Assign the timestamp & the correct height
  \
  `block.time = new Date().getTime().toString().slice(0,-3);`
  \
  `block.height = self.chain.length;`
- Create the block hash and push the block into the chain array.
  \
  `block.hash = SHA256(JSON.stringify(block)).toString();`

## Modify 'requestMessageOwnershipVerification(address)' to allow you to request a message that you will use to sign it with your Bitcoin Wallet (Electrum or Bitcoin Core)

- must return a Promise that will resolve with the message to be signed

```
return new Promise((resolve) => {
            // <WALLET_ADRESS>:${new Date().getTime().toString().slice(0,-3)}:starRegistry
            resolve(address + ':' + new Date().getTime().toString().slice(0,-3) + ':starRegistry');

        });
```

## Modify 'submitStar(address, message, signature, star)' function to register a new Block with the star object into the chain

- must resolve with the Block added or
  reject with an error.
- time elapsed between when the message was sent and the current time must be less that 5 minutes
  \
  `let timeMessage = parseInt(message.split(':')[1]);`
  \
  `let currentTime = parseInt(new Date().getTime().toString().slice(0, -3));`
  \
  `if (timeMessage > currentTime - 300000){`
- must verify the message with wallet address and signature: bitcoinMessage.verify(message, address, signature) and must create the block and add it to the chain if verification is valid

````
if (bitcoinMessage.verify(message, address, signature)) {
                    // create the block and add it to the chain
                    let block = new BlockClass.Block({"owner":address, "star":star});
                    await self._addBlock(block);
                    resolve(block);
                }
```
````

## Modify the 'getBlockHeight(hash)' function to retrieve a Block based on the hash parameter

- must return a Promise that will resolve with the Block

```
return new Promise((resolve, reject) => {
            let block = self.chain.filter(p => p.height === height)[0];
            if(block){
                resolve(block);
            } else {
                resolve(null);
            }
```

## Modify the 'getStarsByWalletAddress (address)' function to return an array of Stars from an owners collection

- must return a Promise that will resolve with an array of the owner address' Stars from the chain

```
return new Promise((resolve, reject) => {
            self.chain.forEach(async(block) => {
                let blockData = await block.getBData();
                if (blockData.owner === address) {
                    stars.push(blockData);
                }
            });
            resolve(stars);
        });
```

## Modify the 'validateChain()' function

- must return a Promise that will resolve with the list of errors when validating the chain.
  \
  `let errorLog = [];`
  `return new Promise(async (resolve, reject) => { ... }`
- must validate each block using validateBlock()
  `self.chain.forEach(async(block) => { ... }`

- Each Block should check with the previousBlockHash
  `if (previousBlock.hash != block.previousBlockHash){ ... }`
- execute the validateChain() function every time a block is added
- create an endpoint that will trigger the execution of validateChain()

# Test your App functionality

Use 'POSTMAN'

- must use a GET call to request the Genesis block
  ![Request: http://localhost:8000/block/height/0 ](.\images\1-block_0_genesisblock.png)
  \
- must use a POST call to requestValidation
  ![Request: http://localhost:8000/requestValidation ](.\images\2-requestValidation.png)

- must sign message with your wallet
  ![Use the Wallet to sign a message](.\images\3-signMessage.png)

- must submit your Star
  ![Request: http://localhost:8000/submitstar](.\images\4-submitStar.png)

- must use GET call to retrieve starts owned by a particular address
  ![Request: http://localhost:8000/blocks/<WALLET_ADDRESS>](.\images\5-getStarsByWalletAddress.png)

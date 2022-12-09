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
  ![Request: http://localhost:8000/block/height/0 ](images\1-block_0_genesisblock.png)
  \
- must use a POST call to requestValidation
  ![Request: http://localhost:8000/requestValidation ](images\2-requestValidation.png)

- must sign message with your wallet
  ![Use the Wallet to sign a message](images\3-signMessage.png)

- must submit your Star
  ![Request: http://localhost:8000/submitstar](images\4-submitStar.png)

- must use GET call to retrieve starts owned by a particular address
  ![Request: http://localhost:8000/blocks/<WALLET_ADDRESS>](images\5-getStarsByWalletAddress.png)

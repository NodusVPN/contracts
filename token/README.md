# Install, deploy, verify

## Install
Install all dependencies:

    npm i 

Edit .env file:

    // mumbai, polygon api_url
    MUMBAI_API_URL="https://polygon-mumbai.g.alchemy.com/v2/LXjRVkzYb79oTKVQ23N2Lr6IBVyAzrw1​"
    POLYGON_API_URL="​https://api.polygonscan.com​"

    MUMBAI_PRIVATE_KEY="" // Private key in the test network
    POLYGON_PRIVATE_KEY="" // Private key in the main network

    MUMBAI_ETHERSCAN_API_KEY="V6QAZ2H9U11FB32GBSAX333E9DUIJ9MCYE" // etherscan api key
    POLYGON_ETHERSCAN_API_KEY="V6QAZ2H9U11FB32GBSAX333E9DUIJ9MCYE" // etherscan api key

## Deploy
Deploy to the test network:

    npx hardhat run --network mumbai scripts/deploy.ts

Deploy to the main network:

    npx hardhat run --network polygon scripts/deploy.ts

## Verify
Edit scripts/verify.ts:

    const contractAddress = "" // smart contract address.
    
Verify:

    npx hardhat run --network polygon scripts/verify.js

## Tests
Run tests:

    npx hardhat test

# Functions

The smart contract implements the functions of ERC20 standard, ERC20Burnable, Mintable, Pausable extensions are connected.


Read more about eventing, methods, data structure:

https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20

https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Burnable

https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Pausable

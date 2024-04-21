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
    
Edit scripts/config.ts:

    export const blacklist = ""; // blacklist address
    
## Deploy
Deploy to the test network:

    npx hardhat run --network mumbai scripts/deploy.ts

Deploy to the main network:

    npx hardhat run --network polygon scripts/deploy.ts

## Verify
Edit scripts/config.ts:

    const contractAddress = "" // smart contract address

Verify
    
    npx hardhat run --network polygon scripts/verify.js

## Tests
Run tests:

    npx hardhat test

# Functions

    function createVesting(
            uint256 _amount,
            uint256 _firstUnlockPercent,
            uint256 _firstUnlockTimestamp,
            uint256 _unlockPercent,
            uint256 _unlockTime,
            IERC20 _contractAddress
        ) public onlyOwner 
    
Creates a vesting plan. Available only to the owner. 

**_amount** - total tokens for vesting, 

**_firstUnlockPercent** - first unlock percentage, 

**_firstUnlockTimestamp** - first unlock time, 

**_unlockPercent** - subsequent unlocks in percent,

**_unlockTime** - time interval for unlocks,

**_contractAddress** - token address for vesting

    function updateVesting(
            uint256 _vestingId,
            uint256 _amount,
            uint256 _firstUnlockPercent,
            uint256 _firstUnlockTimestamp,
            uint256 _unlockPercent,
            uint256 _unlockTime,
            IERC20 _contractAddress
        ) 
Updates the vesting plan of a _vestingId. Available only to the owner.

    function setVestingHolders(uint256 _vestingId, address[] memory _holders, uint256[] memory _amounts) 
Adds users for the specified vesting. Available only to the owner.

**vestingId** - vesting plan id,

**holders** - holdings list,

**amount** - list of token count values for each holder

    function claim(uint256 _vestingId) public
Function for claiming tokens available to the user.

    function claimable(uint256 _vestingId, address _user, uint256 _claimTimestamp) public view returns (uint256)
Returns the number of tokens available for claiming.

**vestingId** - vesting plan id,

**user** - user address,

**claimTimestamp** - time of claiming

    function amountLocked(uint256 _vestingId, address _user) public view returns (uint256)
Returns the number of tokens in the vesting.

    function amountPaidOut(uint256 _vestingId, address _user) public view returns (uint256) 
Returns the number of tokens already paid out.

    function sweep(IERC20 tokenAddress, address recipient) external onlyOwner
Outputs any token (tokenAddress) from the contract to the recipient address. Available only to the owner.

    function pause() public onlyOwner
Stops all functions of the contract. Available only to the contract owner.

    function unpause() public onlyOwner 
Resumes operation of all contract functions. Available only to the contract owner.

    function transferOwnership(address newOwner) external onlyOwner
Transfers ownership of the contract to the newOwner address. Available only to the contract owner.

# Events

    event CreateVesting(uint256 vestingId, uint256 amount, uint256 firstUnlockPercent, uint256 firstUnlockTimestamp, uint256 unlockPercent, uint256 unlockTime, IERC20 contractAddress);
Creating a new vesting plan.

    event UpdateVesting(uint256 vestingId, uint256 amount, uint256 firstUnlockPercent, uint256 firstUnlockTimestamp, uint256 unlockPercent, uint256 unlockTime, IERC20 contractAddress);
Vesting plan Update.

    event Claim(uint256 vestingId, address _msgSender, uint256 amount);
Withdrawal of funds from the vesting plan

    event Sweep(IERC20 token, address recepient);
Withdrawal of a specific token from a contract.

# Data Structures

        uint256 internal lastVestingId_; // last vesting id
        IBlacklist public blacklist;     // blacklist contract

        // Vesting structure
        struct Vesting {    
            uint256 amount;                 // Number of tokens    
            uint256 firstUnlockPercent;     // Percentage of first unlock
            uint256 firstUnlockTimestamp;   // Time of first unlock
            uint256 unlockPercent;          // Unlock percentage
            uint256 unlockTime;             // Unlock time 
            IERC20 contractAddress;         // Token contract in vesting
        }
    
        mapping(uint256 => Vesting) public vesting;                                 // vestingId => Vesting
        mapping(uint256 => mapping(address => uint256)) internal amountLocked_;     // vestingId => wallet address => amount of blocked tokens 
        mapping(uint256 => mapping(address => uint256)) internal amountPaidOut_;    // vestin


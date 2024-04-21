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

    export const stakeToken = ""; 
    export const blacklist = "";
    export const rewardTokensPerBlock = 100;
    export const depositLock = 86400;
    export const rewardPool = ether("1000000");
    
## Deploy
Deploy to the test network:

    npx hardhat run --network mumbai scripts/deploy.ts

Deploy to the main network:

    npx hardhat run --network polygon scripts/deploy.ts

## Verify
Edit scripts/config.ts:

    const contractAddress = "" // smart contract address.
    
Verify

    npx hardhat run --network polygon scripts/verify.js
    
## Tests
Run tests:

    npx hardhat test

# Functions
    constructor(address _stakeToken, uint256 _rewardTokensPerBlock, uint256 _depositLock, uint256 _rewardPool) {
            stakeToken = IERC20(_stakeToken);
            rewardTokensPerBlock = _rewardTokensPerBlock;
            depositLock = _depositLock;
            rewardPool = _rewardPool;
        }

**stakeToken** - token to stake,


**rewardTokensPerBlock** - number of awards for 1 block,

**depositLock** - deposit freeze time,

**rewardPool** - total reward tokens

    function stake(uint256 _amount) external
    
Transfers the specified number of tokens to the contract address. Initiates the Deposit event.

    function unstake() external

Outputs the body of the user's contribution and rewards. Initiates the Withdraw event.

Requirements: 

- elapsed time from the start of staking is greater than depositLock,

- withdrawal can be greater than zero

    function harvestRewards() public

Withdraws only the rewards, the steaking body stays in the contract. Initiates the HarvestReward event.

    function harvestable(address _stakerAddress, uint256 _blockNumber) public view returns (uint256) 
    
Returns the number of tokens for the reward at the time of a particular block (_blockNumber).

    function setDepositLock(uint256 _depositLock) public onlyOwner
    
Updates the user's deposit freeze time. Available only to the owner.

    function setRewardPerBlock (uint256 _rewardPerBlock) external onlyOwner

Sets a new value for the block reward. Available only to the owner.

    function setRewardPool (uint256 _rewardPool) external onlyOwner

Sets a new value for the number of rewards. Available only to the owner.

    function sweep(IERC20 tokenAddress, address recipient) external onlyOwner

Outputs any token (tokenAddress) from the contract to the recipient address. Available only to the owner of the

    function pause() public onlyOwner

Stops all functions of the contract. Available only to the contract owner.

    function unpause() public onlyOwner 

Resumes operation of all contract functions. Available only to the contract owner.

    function transferOwnership(address newOwner) external onlyOwner

Transfers ownership of the contract to the newOwner address. Available only to the contract owner.

# Events 

    event Deposit(address indexed user, uint256 amount);

Deposit of tokens from a specific user.

    event Withdraw(address indexed user, uint256 amount);

Withdrawal of the user's entire deposit.

    event HarvestRewards(address indexed user, uint256 amount);

Withdrawal of the user's rewards.

     event Sweep(IERC20 token, address recepient);
 
Withdrawal of a specific token from a contract.

# Data Structures

    uint256 private rewardTokensPerBlock; 
    // Number of reward tokens minted per block 10000..00

    uint256 private constant REWARDS_PRECISION = 1e18; 
    // A big number to perform mul and div operations


    // Staking user for a pool
    struct PoolStaker {
           uint256 amount; // The tokens quantity the user has staked.
           uint256 rewardDebt; // The amount relative to accumulatedRewardsPerShare the user can't get as reward
     }

     // Staking pool
     IERC20 stakeToken; // Token to be staked
     uint256 public tokensStaked; // Total tokens staked
     uint256 public lastRewardedBlock; // Last block number the user had their rewards calculated
     uint256 public accumulatedRewardsPerShare; // Accumulated rewards per share times REWARDS_PRECISION

     // Mapping staker address => PoolStaker
     mapping(address => PoolStaker) public poolStakers;

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

    export const token = ""; // voting token address
    export const blacklist = ""; // blacklist contract address

## Deploy
Deploy to the test network:

    npx hardhat run --network mumbai scripts/deploy.ts

Deploy to the main network:

    npx hardhat run --network polygon scripts/deploy.ts

## Verify
Edit scripts/verify.ts:

    const contractAddress = "" // smart contract address

Verify

    npx hardhat run --network polygon scripts/verify.js

## Tests
Run tests:

    npx hardhat test

# Functions

    constructor(IERC20 _token, IBlacklist _blacklist) {
            token = _token;
            blacklist = _blacklist;
        }
        
Constructor:

**token** - voting token,

**blacklist** - blacklist smart contract

---

    function createProposal(
            address _to,
            uint _value,
            string calldata _func,
            bytes calldata _data,
            string calldata _description
        ) external isNotBlackListed returns(bytes32)
    
Creating a new proposal to be voted on:

**_to** - smart contract on which the function should be executed, 

**_value** - value passed with the function, 

**_func** - string with the name of the function, 

**_data** - bytes encoded in ABI encoder data of the function, 

**_description** - arbitrary description of the function

Returns: proposalId in bytes


Example:

    const proposeTx = await dao.createProposal(
                      0x1234...9,
                      10,
                      "pay(string)",
                      ethers.utils.defaultAbiCoder.encode(['string'], ['test']),
                      "Sample proposal"
                    );
                    
---

    function execute(
            address _to,
            uint _value,
            string calldata _func,
            bytes calldata _data,
            bytes32 _descriptionHash
        ) external onlyOwner returns(bytes memory)
    
The fulfillment function of the sentence. The data is similar as in the createProposal function. Only the owner is available for execution.

---

    function lockVote(bytes32 proposalId, uint8 voteType, uint256 amount) external
Block tokens in favor of the proposal.

**proposalId** - proposal id,

**voteType** - where to vote,

**amount** - token amount

**voteType:**

against - 0

for - 1 

abstain - 2

---

    function unlockVote(bytes32 proposalId) external
    
Unlock tokens. Returns tokens that participated in the proposalId vote. Available only if the vote has ended or canceled.

---

    function cancelVote(bytes32 proposalId) external
    
Cancel the vote. Available only for the creator of the vote.

---

    function setDuration (uint256 duration) external onlyOwner
    
Sets the  VOTING_DURATION variable - voting duration in seconds.

---

    function getVote(bytes32 proposalId) public view returns (uint, uint, uint) 
    
Returns the number of votes (against, for, abstain) of proposalId.

---

    function state(bytes32 proposalId) public view returns (ProposalState)
    
Returns the current state of the vote:

    enum ProposalState { Pending, Active, Succeeded, Defeated, Executed, Canceled }

# Events

    event ProposalAdded(bytes32 proposalId);

Proposal Creation.

    event UnlockVote (bytes32 proposalId, address msgSender);

Unlocking voting tokens.

# Data Structures
    
    IBlacklist public blacklist; // blacklist smart contract
    
        //  voting structure
        struct ProposalVote {
            uint againstVotes;                      //  number of tokens "against"
            uint forVotes;                          //  number of "for" tokens
            uint abstainVotes;                      //  number of "abstain" tokens
            mapping(address => bool) hasVoted;      //  address => voted (true/false)
            mapping(address => uint256) amountVote; //  address => amount tokens
            mapping(address => bool) unlockToken;   //  address => unlocked the tokens (true/false)
            
        }
    
        // voting structure
        struct Proposal {
            uint votingStarts;      //  start of voting
            uint votingEnds;        //  end of voting
            bool executed;          //  whether the proposal has been carried
            bool isCanceled;        //  whether the proposal has been completed
            address ownerVotes;     //  voting creator
        }
    
        // voting status
        enum ProposalState { Pending, Active, Succeeded, Defeated, Executed, Canceled }
    
        mapping(bytes32 => Proposal) public proposals;          // proposalId => Proposal
        mapping(bytes32 => ProposalVote) public proposalVotes;  // proposalId => ProposalVote
    
        IERC20 public token;                        // voting token
        uint256 public VOTING_DURATION = 86400;     // voting time in seconds

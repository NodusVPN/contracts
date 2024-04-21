# Install, Deploy, Verify

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

    export const paymentToken = ""; // token for payment
    export const feePercent = 500; // percent fee 500 = 5%
    export const serverAddCost = ether("100"); // adding server fee
    export const feeRecipient = ""; // fee recipient
    export const maxServersPerUser = 100; // max servers per user
    export const maxPackTypesPerServer = 100; // max packtypes per server
    export const maxPacksPerTypes = 100; // max packs per packtypes
    
    export const contractAddress = "" // deployed for verify
    ``
    
## Deploy

Deploy to the test network:

    npx hardhat run --network mumbai scripts/deploy.ts

Deploy to the main network:

    npx hardhat run --network polygon scripts/deploy.ts

## Verify
Edit scripts/verify.ts:

    npx hardhat run --network polygon scripts/verify.ts

## Tests
Run tests:

    npx hardhat test

# Functions

    constructor(
            IERC20 _paymentToken, 
            uint256 _feePercent, 
            uint256 _serverAddCost, 
            address _feeRecipient, 
            uint256 _maxServersPerUser,
            uint256 _maxPackTypesPerServer,
            uint256 _maxPacksPerTypes}

Constructor:

**paymentToken** - token for payment

**feePercent** - percent fee (500 = 5%)

**serverAddCost** -  adding server fee

**feeRecipient** - fee recipient

**maxServersPerUser** - max servers per user

**maxPackTypesPerServer** - max packtypes per server

**maxPacksPerTypes** - max packs per packtypes

---

    function addServer(string memory serverID, AddingPackType[] memory packTypes)

**serverID** - string id server

AddingPackType - Structure for adding a server:

**bool isTime** - TRUE - if the packet is time-limited, FALSE - if by traffic volume

**uint256 size** - packet size

**uint256 step** - stepe of enrollment, periodicity of updating balances

**uint256 price** - total cost of the package

Adds a server with the specified packet types from the PackTypes array, where the provider is the sender of the transaction.

Method transfers funds from the provider's balance for hosting the server to its own balance. Saves the server. Issues a event ServerAdded. Adds all packets and for each added packet issues a event PackTypeAdded;

Exceptions: 

- If the number of created servers at the provider is greater than MAX_SERVERS_PER_USER - Amount servers limit

---

    function editServer(string serverID, AddingPackType[] memory addedPackTypes, uint256[] memory removingPackTypeIDs)

**serverID** - server identifier

**AddingPackType** - Structure for adding a server

**removingPackTypeIDs** - array of packType identifiers to be deleted

Adds packets from the addedPackTypes[] array and deactivates or removes packets with identifiers from the removingPackTypeIDs[] array from the server with identifier serverID.

Method adds all added packet types and for each added packet type issues **event PackTypeAdded**. For each package type to be removed, SC checks if there are active packages of that type: if there are, SC deactivates it (changes attribute isActive=FALSE), and issues an **event PackTypeDeactivated**; otherwise it deletes it and issues the **event PackTypeRemoved**.

Exceptions:

- The user is not the server provider - You are not provider on this server

- If the packet to be deleted does not belong to the server - PackTypes is not include on this server

- If the number of created packets exceeds MAX_PACK_TYPES_PER_SERVER - Amount pack types limit

---

    function removeServer(string serverID)

**serverID** - server identifier

Deletes the specified server with serverID.

Method removes all available packet types, but does not terminate the current packets. If there are still current packets, the SC does not delete the server and returns an error. and Issues the. If successful, it generates **event ServerDeleted**.

Exceptions:

- The user is not the server provider - You are not provider on this server

- If there are active packets - The server has active packages

---

    function buyPack (address providerAddress, string serverID, uint256 packTypeID)

**providerAddress** - provider address

**serverID** - server identifier

**packTypeID** - Identifier of the package to be purchased

If the package type to be purchased is traffic-based, the SC transfers the funds for the package purchase from the user's address to the Provider's address, otherwise to its own address. SC saves the purchased package. SC issues an **event PackPurchased**

Exceptions:

- If he can't find the server - Server ID not found

- If the packet to be deleted does not belong to the server - PackTypes is not include on this server

- If the server is not active - Server is not active

- If the pack type is not active - Pack Type is not active

If the number of purchase at the provider is greater than MAX_SERVERS_PER_USER - Amount servers limit

---

    function claimTimePackPayment(uint256 packID, uint256 packTypeID, string serverID)

**packID** - Identifier of the pack to be purchased

**packTypeID** - Identifier of the package to be purchased

**serverID** - server identifier

Collect the payment for the used time package from the contract balance.

Method verifies that the provider is indeed the provider of this packet. 
Verifies that the packet is time-limited and has been exhausted. 
SC transfers to the address of the provider the payment for the used package, commission to the address specified as feeRecipient. 
SC deletes the used package. 
SC issues a **event TimePackClaimed**. 
If it was the last package of inactive type (isActive = false, there are no other packages of this type that are not exhausted), SC deletes this inactive package type and issues **event PackTypeRemoved**

Exceptions:

- If it's not the provider that's trying to collect - Only provider can claim

- If this provider does not have the specified server - Server ID is not exists

- If invalid packID or packTypeID is specified - Invalid packtypeid or packid

- If the packet has not expired - Pack is not ended

- If the attempt to collect a packet is not time-limited - Only time pack can claim

---

    function cancelTimePack (address providerAddress, string serverID, uint256 packTypeID, uint256 packID)

**providerAddress** - provider address

**serverID** - server identifier

**packTypeID** - Identifier of the package to be purchased

**packID** - Identifier of the pack to be purchased

Cancels a purchased time-limited package (subscription).

Method verifies that the actor has purchased this package or is the provider of this package.  
SC calculates the term balance, and based on the term balance calculates how much to transfer to the user and provider and calculates the platform's commission. 
SC transfers: to the user's address the balance for the unused portion of the package, to the provider's address the payment for the used part of the package, commission to the address specified as feeRecipient. 
SC deletes the canceled package. 
SC issues a **event TimePackCanceled**
If it was the last packet of an inactive packet type, the SC deletes this inactive packet type and issues a **event PackTypeRemoved**.

Exceptions:

- Checks that the sender of the transaction is a buyer or provider - Your not provider or buyer

- If the package has already expired - Pack is ended

- If the specified ID server does not exist - Server ID not found

- If pack type are not included in the server - PackTypes is not include in server

- If the package is not time-limited - Pack is not TimePack

---

    function setFeePercent (uint256 newFeePercent)

Available only to the owner. 
Sets the commission for using the service in percent. 

Example: 
- 5% = 500 
- 10% = 1000
- 
Generates an **event FeeChanged**

---

    function setFeeRecipient(address newRecipient)
    
Available only to the owner. 
Sets up a new recipient of service commissions.

Generates an **event FeeRecipientChanged**

---

    function setServerAddCost(uint256 newServerAddCost) 
    
Available only to the owner. 
Modifies the server creation fee. Specified in decimals(18).

Generates an **event ServerAddCostChanged**

---

    function setMaxServersPerUser (uint256 maxServers)
    
Available only to the owner. 
Sets the maximum number of servers per 1 provider.

Generates an **event SetMaxServersPerUser**

---

    function setMaxPackTypesPerServer (uint256 maxPackTypes)
    
Available only to the owner. 
Sets the maximum number of pack types for a server.

Generates an **event SetMaxPackTypesPerServer**

---

    function setMaxPacksPerPackTypes (uint256 maxPacks)
    
Available only to the owner. 
Sets the maximum number of packs for one pack types.

Generates an **event SetMaxPacksPerPackTypes**

---

    function setPaymentToken(IERC20 token)
    
Available only to the owner. 
Sets a new token for payment.

Generates an **event SetPaymentToken**

---

    providerDetailsOf(address provider)
    
View-function.
Returns all provider data in the ProviderServers structure.

---

    serverDetailsOf (string serverID)
    
View-function.
Returns all information about the server by its ID from the Server structure

---

    packTypeDetailsOf(uint256 packTypeID)
    
View-function.
Returns all information about the packType by its ID from the PackType structure

---

    packDetailsOf(uint256 packID)
    
View-function.
Returns all information about the pack by its ID from the Pack structure.

---

    function getPackTypesFromServer (uint256 serverID) public view returns(uint256[] memory) 
    
Returns an array of all created packTypes by serverID.

---

    function getServersFromProvider (address provider) public view returns(uint256[] memory)
    
Returns an array of all created servers by provider address.

---

    function getPacksFromPackTypes (uint256 packType) public view returns(uint256[] memory)
    
Returns an array of all created packs by packType.

# Events

        // Adding a server
        event ServerAdded(string serverID, address provider);
        
        // Deleting a server
        event ServerDeleted(string serverID);
        
        // Adding pack type
        event PackTypeAdded(uint256 packTypeID, string serverID, bool isTime, uint256 size, uint256 step, uint256 price);
        
        // Deactivation of pack type
        event PackTypeDeactivated(uint256 packTypeID);
        
        // Deleting pack type
        event PackTypeRemoved(uint256 packTypeID);
        
        // Buying a package
        event PackPurchased(uint256 packID, uint256 packTypeID, address userAddress);
        
        // Claimed time pack
        event TimePackClaimed(uint256 packID);
        
        // Cancel time pack
        event TimePackCanceled(uint256 packID, address userAddress);
        
        // Change of commission
        event FeeChanged(uint256 newFee);
        
        // Changing the recipient of the commission
        event FeeRecipientChanged(address newRecipient);
        
        // Changing the commission for server creation
        event ServerAddCostChanged(uint256 newServerAddCost);
        
        // Changing the maximum number of servers at the provider
        event SetMaxServersPerUser (uint256 maxServers);
        
        // Changing the maximum number of pack types on the server
        event SetMaxPackTypesPerServer (uint256 maxPackTypes);
        
        // Changing the maximum number of packets in pack type
        event SetMaxPacksPerPackTypes (uint256 maxPacks);
        
        // Set token for payment
        event SetPaymentToken(IERC20 token);
        
        // Output token from smart contract address
        event Sweep(IERC20 token, address recipient);


# Data Structures

        uint256 public serverAddCost;    // added server fee
        address public feeRecipient;     // fee recipient
    
        IERC20 public paymentToken;    // payment token
    
        uint256 public totalPackTypes;   // total number of pack types added   
        uint256 public totalServers;     // total number of servers added
        uint256 public totalPack;        //total number of packs added
    
        uint256 public MAX_SERVERS_PER_USER;    // maximum servers per user
        uint256 public MAX_PACK_TYPES_PER_SERVER; // maximum pack types per server
        uint256 public MAX_PACKS_PER_PACK_TYPES;    // maximum packs per pack types
    
        uint256 private constant BP = 100_00; // base percent. default = 100%
        uint256 public FEE_PERCENT = 5_00;    // fee percent. default = 5%
        
        // Server structure at the provider
        struct ProviderServers {    
            string [] servers;    // server ID list
            uint256 serversAmount; // number of servers created
        }
        
       // specific server structure
        struct Server {
            bool isActive;        // active: true / false
            uint256[] packTypes;  // pack types ID list
            uint256 packTypesAmount; // number of packrtpes added
        }
    
        // Structure for adding a server
        struct AddingPackType {
            //TRUE - if the packet is time-limited, FALSE - if by traffic volume.
            bool isTime;
            // Packet size: the duration of the term if the packet is time-limited, 
            // or the amount of traffic if the packet is traffic-limited.
            uint256 size;
            // Step of enrollment, periodicity of updating balances.
            // Time unit, if isTime==TRUE,
            // traffic unit, if isTime==FALSE.
            uint256 step;
            // Total cost of the package.
            uint256 price;
        }
    
        struct PackType {
            // active: true / false
            bool isActive;
            
            //TRUE - if the packet is time-limited, FALSE - if by traffic volume.
            bool isTime;
            
            // Packet size: the duration of the term if the packet is time-limited, 
            // or the amount of traffic if the packet is traffic-limited.
            uint256 size;
            
            // Step of enrollment, periodicity of updating balances.
            // Time unit, if isTime==TRUE,
            // traffic unit, if isTime==FALSE.
            uint256 step;
            
            // Total cost of the package.
            uint256 price;
            
            // An array of packets of this type ever created on this server.
            uint256[] packs;
            
            // The number of packages of a given type purchased.
            uint256 packAmount;
        }
    
        struct Pack {
            address userAddress;     // buyer address 
            address providerAddress; // provider address
            uint256 used;            // Number of steps enrolled.
            uint256 purchaseTimestamp;    // purchase timestamp
            uint256 endTimestamp;    // packet end timestamp
            uint256 pricePerStep;    // price per enrollment step
            uint256 step;            // Step of enrollment
            uint256 priceAmount;     // Cost of the package.
            uint256 feeAmount;        // Cost of the fee
        }

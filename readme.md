# Contract Review of the Aavegotchi Alchemica Facet

## The Aavegotchi game

The aim of the game is to obtain alchemica, which comes in four types: FUD, FOMO, KEK, and ALPHA. Players can acquire alchemica through farming, channeling, and scavenging.

Here are some details about alchemical channeling:

- Aavegotchis can channel alchemica by visiting an altar and clicking the "Channel Alchemica" action button. Channeling rewards Aavegotchis with alchemica, some of which spills onto nearby land, allowing players to scavenge it before others.

- Alchemica received during channeling is influenced by the Aavegotchi's kinship level. Higher kinship levels result in more alchemica.

- The spillover rate determines how much alchemica is distributed to the Gotchis and how much spills over into nearby land. Higher-level altars have lower spillover rates and shorter cooldown times between channeling.

- Guilds play a significant role in channeling. Guilds have Gotchi Lodges with varying sizes and the ability to decorate them with furniture and display NFTs. Players enter a guild using a guild crest, which is an NFT acting as a key and provided by guild owners to members.

- Channeling with a guild has benefits, including a sociable experience and increased alchemica rewards. The more Aavegotchis in the guild and their rarity, the more alchemica rains down.

- Wearable crests for guilds are obtained through bid-to-earn auctions using glitter tokens.

- Glitter tokens have utility and are distributed over 30 years, with 10% in the first year and halving every four years. Players can earn glitter by providing liquidity to the Aavegotchi decentralized exchange (GAX) with their ghost FUD, FOMO, KEK, and ALPHA tokens.

- Glitter can be used to speed up installation build times, with each glitter reducing build time by one block.

- The GotchiVerse Bible mentions wearable abilities

The information provides insights into how alchemical channeling, guilds, glitter tokens, and wearables function in the Aavegotchi game.

<br>

<br>

## Link to Contract

[AlchemicaFacet.sol](https://github.com/aavegotchi/aavegotchi-realm-diamond/blob/master/contracts/RealmDiamond/facets/AlchemicaFacet.sol)

<br>

<br>

## Contract Review

- The contract is inheriting the contract Modifiers from the library/AppStorage.sol.
- The AppStorage.sol has 12 unique constant variables, a struct Parcel, a struct AppStorage that contains the storage variables to store, and the library LibAppStorage sets the appstorage in a slot. Contract Modiifier sets an onlyParcelOwner which can call a certain tokenId. The modifier onlyOwner ensures that it is the person that deployed the Diamond.

<br>

<br>

## State Variables

- A constant bp of 100 ether
- There are five various events that the contract uses subgrapgh to emit.

  ```event StartSurveying(uint256 _realmId, uint256 _round);

  event ChannelAlchemica(
    uint256 indexed _realmId,
    uint256 indexed _gotchiId,
    uint256[4] _alchemica,
    uint256 _spilloverRate,
    uint256 _spilloverRadius
  );

  event ExitAlchemica(uint256 indexed _gotchiId, uint256[] _alchemica);
  event SurveyingRoundProgressed(uint256 indexed _newRound);
  event TransferTokensToGotchi(address indexed _sender, uint256 indexed _gotchiId, address _tokenAddresses, uint256 _amount);
  ```

```

```

## Function Definitons

**Function Name:** isSurveying

- **Visibility:** External
- **Return Type:** Boolean (true if the realm is surveying, false otherwise)
- **Parameters:** `_realmId` - The unique identifier of the realm being queried.

**Purpose:** This function checks the status of surveying for a given realm. If it returns true, it means that the specified realm is currently in the surveying state. This information is useful for external applications or functions that need to determine the state of a particular realm.

<br>

<br>

**Function Name:** startSurveying

- **Visibility:** External (can be called from outside the contract)

**Parameters:**

- `_realmId` - The unique identifier of the realm to start surveying.

**Modifiers:**

- `onlyParcelOwner(_realmId)`: Ensures that only the owner of the specified realm can call this function.
- `gameActive`: Requires that the game is in an active state to proceed.

**Requirements:**

- The current round of the realm must be less than or equal to the global surveying round. This ensures that the realm is surveying in the correct order.
- The realm must have an Altar equipped (altarId > 0) before starting the survey.
- The realm should not already be in the surveying state (!s.parcels[_realmId].surveying).

**Functionality:**

If all requirements are met, the function performs the following actions:

- Sets the surveying state of the specified realm to true, indicating that it has started surveying.
- Calls the `drawRandomNumbers` function to obtain random numbers for the realm's current round.
- Emits a `StartSurveying` event, providing information about the realm and the current round.

**Summary:**
This function, accessible to parcel owners, is used to initiate the surveying process for their realms. It enforces specific conditions, ensuring that the realm is eligible to start the survey, and then sets the surveying state, obtains random numbers, and emits an event to track the survey's progress.

<br>

<br>

**Function Name:** drawRandomNumbers

- **Visibility:** Internal (can only be called within the contract)

**Parameters:**

- `_realmId`: The unique identifier of the realm for which random numbers are requested.
- `_surveyingRound`: The round of surveying for which random numbers are being requested.

**Functionality:**

This function performs the following tasks:

- Initiates a request for random numbers from a Chainlink VRF V2 (Verifiable Random Function) service.
- The request includes various configuration parameters, such as the key hash, subscription ID, request confirmations, callback gas limit, and the number of random words (numbers) to be generated.
- The Chainlink VRF service processes this request asynchronously.
- The request ID returned by the VRF service is associated with the `_realmId` and `_surveyingRound` in the contract's storage.

**Summary:**
The `drawRandomNumbers` function is utilized to request random numbers from the Chainlink VRF V2 service for a specific realm and surveying round. It triggers the request and records essential information in the contract's storage for subsequent retrieval and processing.

<br>

<br>

**Function Name:** getTotalAlchemicas

- **Visibility:** View (read-only function)

**Return Type:** Two-dimensional array

**Functionality:**

This function allows querying details about all total alchemicas present in the contract. It returns a two-dimensional array with a specific structure:

- The array has 4 rows, each corresponding to one of the four alchemica types.
- It consists of 5 columns, each corresponding to a different quantity or total for those alchemica types.

**Summary:**

The `getTotalAlchemicas` function is a read-only function in the AlchemicaFacet contract, offering access to the stored data representing total alchemica values. Users can retrieve this information without making any changes to the contract's state.

<br>

<br>

**Function Name:** getRealmAlchemica

- **Visibility:** View (read-only function)

**Parameters:**

- `_realmId`: The identifier of the parcel to be queried.

**Return Type:** Array with four elements

**Functionality:**

This function allows users to query details about the remaining alchemica in a specific parcel, specified by the `_realmId`. It returns an array with four elements, where each element represents the remaining quantity of a different type of alchemica: FUD, FOMO, ALPHA, and KEK.

**Summary:**

The `getRealmAlchemica` function is a read-only function in the AlchemicaFacet contract, enabling users to retrieve information about the quantities of different alchemicas remaining in a particular parcel without making any changes to the contract's state.

<br>

<br>

**Function Name:** getParcelCurrentRound

- **Visibility:** View (read-only function)

**Parameters:**

- `_realmId`: The identifier of the parcel to be queried.

**Return Type:** uint256 (the current round of surveying)

**Functionality:**

This function allows users to query the current round of surveying for a specific parcel, identified by the `_realmId`. It returns a single `uint256` value, representing the current round of surveying for the specified parcel.

**Summary:**

The `getParcelCurrentRound` function is a read-only function in the AlchemicaFacet contract, offering a way for users to retrieve the current surveying round of a specific parcel without altering the contract's state.

<br>

<br>

**Function Name:** progressSurveyingRound

- **Visibility:** External (can only be called by the contract owner)

**Functionality:**

This external function is exclusively accessible to the contract owner. When called, it performs the following tasks:

- Increments the `surveyingRound` by one, indicating the progression of surveying rounds.
- Emits a `SurveyingRoundProgressed` event to notify external observers about the updated surveying round value.

**Summary:**

The `progressSurveyingRound` function enables the contract owner to advance the surveying rounds, which may have implications for the surveying process within the contract. This function is accessible only to the contract owner which is the person that deployed the Diamond.

<br>

<br>

**Function Name:** getRoundAlchemica

- **Visibility:** External View (can be called by anyone)

**Parameters:**

- `_realmId`: The identifier of the parcel.
- `_roundId`: The identifier of the surveying round.

**Return Type:** Array of uint256 values

**Functionality:**

This external view function takes two parameters, `_realmId` (parcel identifier) and `_roundId` (surveying round identifier), and provides the following functionality:

- It retrieves information about the alchemica gathered during the specified surveying round for the given parcel.
- The gathered alchemica values are returned as an array of `uint256` values, representing the quantities of different types of alchemica collected during that round.

**Summary:**

The `getRoundAlchemica` function offers transparency and accessibility to users or external systems, enabling them to retrieve historical data regarding alchemica gathered during specific surveying rounds for particular parcels.

<br>

<br>

**Function Name:** getRoundBaseAlchemica

- **Visibility:** External View (accessible to anyone)

**Parameters:**

- `_realmId`: The identifier of the parcel.
- `_roundId`: The identifier of the surveying round.

**Return Type:** Array of uint256 values

**Functionality:**

This external view function takes two parameters, `_realmId` (parcel identifier) and `_roundId` (surveying round identifier), and provides the following functionality:

- It retrieves information about the base alchemica gathered during the specified surveying round for the given parcel.
- The gathered base alchemica values are returned as an array of `uint256` values, representing the quantities of different types of base alchemica collected during that round.

**Summary:**

The `getRoundBaseAlchemica` function offers transparency and accessibility for users and external systems to retrieve historical data regarding base alchemica gathered during specific surveying rounds for particular parcels.

<br>

<br>

**Function Name:** setVars

- **Visibility:** External (accessible only to the contract owner)

**Parameters:**

- `_alchemicas`: An array of arrays containing quantities of alchemicas available.
- `_boostMultipliers`: An array of multipliers applied to each parcel.
- `_greatPortalCapacity`: An array specifying individual alchemica capacity of the great portal.
- `_installationsDiamond`: Address of the installations diamond.
- `_vrfCoordinator`: Address of the Chainlink VRF Coordinator.
- `_linkAddress`: Address of the LINK token.
- `_alchemicaAddresses`: An array of addresses representing the four alchemica tokens.
- `_gltrAddress`: Address of the GLTR token.
- `_backendPubKey`: Realm (Gotchiverse) backend public key.
- `_gameManager`: Address of the game manager.
- `_tileDiamond`: Address of the tile diamond.
- `_aavegotchiDiamond`: Address of the Aavegotchi diamond.

**Functionality:**

This external function is exclusively accessible to the contract owner and serves the following purpose:

- It allows the contract owner to configure various parameters and addresses for the Alchemica system.
- The function iterates through the provided arrays and assigns their values to the corresponding storage variables within the contract.

**Summary:**

The `setVars` function acts as a configuration method for initializing and updating critical parameters and addresses within the Alchemica system. This function is primarily used by the contract owner to ensure that the system operates with the correct values and connections.

<br>

<br>

**Function Name:** setTotalAlchemicas

- **Visibility:** External (accessible only to the contract owner)

**Parameters:**

- `_totalAlchemicas`: An array of arrays containing quantities of alchemicas.

**Functionality:**

This external function is exclusively accessible to the contract owner and serves the following purpose:

- It allows the contract owner to update the configuration for the total alchemicas available within the Alchemica system.
- The function iterates through the provided arrays and assigns their values to the corresponding storage variables in the contract.

**Summary:**

The `setTotalAlchemicas` function offers a means for the contract owner to modify the configuration of total alchemicas, ensuring that the Alchemica system operates with the correct quantities of alchemicas for different types and levels. This ability to adjust alchemica quantities is crucial for maintaining and adapting the system's behavior as needed.

<br>

<br>

**Function Name:** getAvailableAlchemica

**Parameters:**

- `_realmId`: The identifier of the parcel to query.

**Return Type:**

- `_availableAlchemica`: An array of `uint256` values representing the available quantities of different types of alchemica.

**Functionality:**

This public view function serves the following purpose:

- It allows users to query information about the available alchemica in a specific parcel, specified by the `_realmId`.
- It uses a loop to iterate through the four types of alchemica (FUD, FOMO, ALPHA, KEK), and for each type, it calls another function, `LibAlchemica.getAvailableAlchemica`, to determine the available quantity for that particular type.

**Summary:**

The `getAvailableAlchemica` function simplifies the process of obtaining available alchemica values for different types within a specific parcel. This information is crucial for various actions and decisions within the Alchemica system.

<br>

<br>

**Function Name:** calculateTransferAmounts

**Parameters:**

- `_amount`: The total amount of alchemica to be allocated.
- `_spilloverRate`: A rate determining the spillover amount.

**Return Type:**

- A `TransferAmounts` struct containing two `uint256` values: `owner` and `spill`.

**Functionality:**

This internal pure function serves the following purpose:

- It calculates the amount of alchemica allocated to the owner and the spillover, based on the provided total amount and the spillover rate.
- It employs a mathematical formula to determine the owner's allocation by subtracting the spillover amount from the total amount.
- The spillover amount is computed by multiplying the total amount by the spillover rate and dividing by a constant (bp).

**Summary:**

The `calculateTransferAmounts` function streamlines the allocation calculation process for determining how alchemica should be distributed between the owner and the spillover. It ensures precise allocation according to the defined mathematical formula.

<br>

<br>

**Function Name:** lastClaimedAlchemica

**Parameters:**

- `_realmId`: The identifier of the parcel (realm) for which you want to know the timestamp of the last claimed alchemica.

**Return Type:**

- `uint256`: A timestamp representing the last time alchemica was claimed for the specified parcel.

**Functionality:**

This external view function serves the following purpose:

- It retrieves and returns the timestamp indicating when alchemica was last claimed for the specified parcel (realm).
- Users can utilize this information to keep track of the timing of alchemica claims for the parcel.

**Summary:**

The `lastClaimedAlchemica` function promotes transparency by enabling users to access historical data about the most recent alchemica claim for a particular parcel (realm).

<br>

<br>

**Function Name:** claimAvailableAlchemica

**Parameters:**

- `_realmId`: Identifier of the parcel from which alchemica is being claimed.
- `_gotchiId`: Identifier of the Aavegotchi used for alchemica collection.
- `_signature`: Message signature for backend validation.

**Functionality:**

This external function offers the following functionality:

- It first verifies the authenticity of a provided digital signature to ensure the legitimacy of the alchemica claim request.
- The function checks whether the caller (msg.sender) possesses the "Empty Reservoir Access Right" for the specified parcel and Aavegotchi.
- If the signature and access rights are confirmed as valid, the function permits the caller to claim available alchemica for the specified parcel using their Aavegotchi.

**Summary:**

The `claimAvailableAlchemica` function ensures a secure and authorized process for claiming available alchemica from a parcel, utilizing a specific Aavegotchi NFT, while also validating the request's authenticity.

<br>

<br>

**Function Name:** getHarvestRates

**Parameters:**

- `_realmId`: Identifier of the parcel for which the harvest rates are being queried.

**Functionality:**

This external view function performs the following tasks:

- It initializes an array called `harvestRates` to store the harvest rates for the different types of alchemica (four in total).
- The function iterates over the alchemica types (indexed from 0 to 3) and retrieves the corresponding harvest rate values for the specified parcel from the contract's storage.
- Finally, it returns the `harvestRates` array, which contains the harvest rates for the four alchemica types.

**Summary:**

The `getHarvestRates` function offers a convenient means to query and access the harvest rates of alchemica associated with a specific parcel, enhancing transparency and accessibility for users and external systems.

<br>

<br>

**Function Name:** getCapacities

**Parameters:**

- `_realmId`: Identifier of the parcel for which the alchemica capacities are being queried.

**Functionality:**

This external view function performs the following tasks:

- It initializes an array called `capacities` to store the total alchemica capacities for the different types of alchemica (four in total).
- The function iterates over the alchemica types (indexed from 0 to 3) and calculates the total alchemica capacity for each type using a helper function from the `LibAlchemica` library.
- Finally, it returns the `capacities` array, which contains the total alchemica capacities for the four alchemica types in the specified parcel.

**Summary:**

The `getCapacities` function provides a straightforward method to retrieve and access the total alchemica capacities associated with a specific parcel, enhancing transparency and accessibility for users and external systems.

<br>

<br>

**Function Name:** getTotalClaimed

**Parameters:**

- `_realmId`: Identifier of the parcel for which the total claimed alchemica is being queried.

**Functionality:**

This external view function performs the following tasks:

- It initializes an array called `totalClaimed` to store the total alchemica claimed for the different types of alchemica (four in total).
- The function iterates over the alchemica types (indexed from 0 to 3) and calculates the total alchemica claimed for each type using a helper function from the `LibAlchemica` library.
- Finally, it returns the `totalClaimed` array, which contains the total alchemica claimed for the four alchemica types in the specified parcel.

**Summary:**

The `getTotalClaimed` function offers a straightforward method to access information about the total alchemica claimed for each alchemica type in a specific parcel. This data is readily available and accessible for users and external systems.

<br>

<br>

**Function Name:** channelAlchemica

**Parameters:**

- `_realmId`: Identifier of the parcel from which alchemica is being channeled.
- `_gotchiId`: Identifier of the parent ERC721 Aavegotchi to which alchemica is being channeled.
- `_lastChanneled`: The last time alchemica was channeled in this `_realmId`.
- `_signature`: A message signature used for backend validation.

**Functionality:**

This complex external function provides various features and checks for secure alchemica channeling. The process includes:

- Validation to ensure the specified Aavegotchi is not listed for lending, and it reduces the kinship of the Aavegotchi.
- Access rights verification, including checking the last channeled time and enforcing a limit of one channeling every 24 hours.
- Validation of the Altar's level, ensuring it's equipped and calculating the time interval for channeling based on Altar level.
- Security through backend signature validation.
- Determining the spillover rate and radius for channeling.
- Calculating channel amounts based on kinship and alchemica type.
- Minting or transferring alchemica tokens based on the Great Portal balance and capacity.
- Updating the latest channeling timestamps.
- Emitting an event to notify the action of channeling alchemica.

**Summary:**

The `channelAlchemica` function empowers parcel owners to channel alchemica to their Aavegotchi and the Great Portal, incorporating various checks to ensure secure, rule-compliant, and mechanically sound alchemica management in the game. This function plays a critical role in the game's alchemica resource management.

<br>

<br>

**Function Name:** getLastChanneled

- **Visibility:** Public view (accessible to anyone)

**Parameters:**

- `_gotchiId`: Identifier of the parent ERC721 Aavegotchi.

**Return Type:**

- `uint256` (last channeling timestamp)

**Functionality:**

This public view function serves the following purpose:

- It allows anyone to retrieve the timestamp of the last channeling action performed by a specific Aavegotchi.

**Summary:**

The `getLastChanneled` function offers transparency and accessibility by enabling anyone to access the last timestamp when a specific Aavegotchi performed a channeling action. This information is crucial for tracking and managing Aavegotchi interactions within the game.

<br>

<br>

**Function Name:** getParcelLastChanneled

- **Visibility:** Public view (accessible to anyone)

**Parameters:**

- `_parcelId`: Identifier of the ERC721 parcel.

**Return Type:**

- `uint256` (last channeling timestamp)

**Functionality:**

This public view function serves the following purpose:

- It enables anyone to retrieve the timestamp of the last channeling action performed on a specific ERC721 parcel.

**Summary:**

The `getParcelLastChanneled` function provides transparency and accessibility to the last timestamp when channeling occurred on a particular parcel. This information is essential for tracking and managing interactions involving ERC721 parcels within the game.

<br>

<br>

**Function Name:** batchTransferAlchemica

- **Visibility:** External (accessible to anyone)

**Parameters:**

- `_targets`: An array of target addresses.
- `_amounts`: A nested array representing the amounts to transfer. The inner array order for `_amounts` is FUD, FOMO, ALPHA, KEK.

**Functionality:**

This external function serves the following purpose:

- It simplifies the process of batch transferring alchemica tokens to multiple target addresses.
- The function checks for matching array lengths between `_targets` and `_amounts`, and it transfers the specified amounts of different alchemica tokens to the respective target addresses.

**Summary:**

The `batchTransferAlchemica` function streamlines the process of batch transferring alchemica tokens to multiple recipients, ensuring the proper allocation of different alchemica types to the designated target addresses.

<br>

<br>

**Function Name:** batchTransferTokensToGotchis

- **Visibility:** External (accessible to anyone)

**Parameters:**

- `_gotchiIds`: An array of Gotchi IDs (identifiers).
- `_tokenAddresses`: An array of token addresses for transfer.
- `_amounts`: A nested array representing the amounts to transfer to each Gotchi.

**Functionality:**

This external function serves the following purpose:

- It simplifies the process of batch transferring tokens to multiple Aavegotchis (Gotchi NFTs).
- The function checks for matching array lengths between `_gotchiIds`, `_tokenAddresses`, and `_amounts`, and it transfers the specified token amounts to the respective Aavegotchis.
- Additionally, it emits a `TransferTokensToGotchi` event for each transfer.

**Summary:**

The `batchTransferTokensToGotchis` function streamlines the process of batch transferring tokens to multiple Aavegotchis, ensuring that the designated token amounts are correctly distributed to each Gotchi.

<br>

<br>

**Function Name:** setChannelingLimits

- **Visibility:** External (accessible to anyone)

**Parameters:**

- `_altarLevel`: An array of altar levels.
- `_limits`: An array of time limits (in seconds) corresponding to each altar level.

**Functionality:**

This owner-only function allows the owner to change the channeling limits for altars of different levels. It verifies that the array lengths of `_altarLevel` and `_limits` match and assigns the specified time limits to their respective altar levels in the storage.

**Summary:**

The `setChannelingLimits` function enables the owner to modify the channeling time limits for different altar levels, providing flexibility in adjusting the channeling frequency for various altars.

<br>

<br>

**Function Name:** floorSqrt

- **Visibility:** Internal (only accessible within the contract)

**Parameters:**

- `n`: The input number for which the floor square root is to be calculated.

**Return Type:**

- `uint256` (the floor square root of the input)

**Functionality:**

This internal pure function calculates the floor square root of a given input number using an iterative algorithm. It returns the floor square root as a `uint256`.

**Summary:**

The `floorSqrt` function is used to calculate the floor square root of a number and is accessible only within the contract. It's useful for various calculations within the contract logic.

<br>

<br>

**Function Name:** \_batchTransferTokens

- **Visibility:** Internal (only accessible within the contract)

**Parameters:**

- `_tokens`: An array of token addresses to be transferred.
- `_amounts`: An array of token amounts to transfer.
- `_to`: The target address to receive the tokens.

**Functionality:**

This internal function serves the following purpose:

- It is designed for batch token transfers, allowing the transfer of multiple tokens from the sender's address to a specified recipient.
- The function enforces that the arrays `_tokens` and `_amounts` have the same length to prevent mismatches.
- If any transfer within the batch fails, the function reverts the transaction and provides an error message that may include the failing token's name (if available), aiding in identifying the problematic token.

**Summary:**

The `_batchTransferTokens` function offers an efficient means to perform batch transfers of multiple tokens from the sender's address to a specified recipient. It also handles potential transfer failures gracefully, reverting the transaction with an error message for improved diagnostic purposes.

<br>

<br>

**Function Name:** batchTransferTokens

- **Visibility:** External (accessible to anyone)

**Parameters:**

- `_tokens`: A nested array of token addresses to be transferred (arrays of arrays).
- `_amounts`: A nested array of token amounts to transfer (arrays of arrays).
- `_to`: An array of target addresses to receive the tokens.

**Functionality:**

This external function serves the following purpose:

- It allows batch transfers of multiple tokens to multiple recipients.
- The function checks for mismatches in the array lengths and ensures that the corresponding `_tokens` and `_amounts` arrays have the same length.
- For each recipient address in the `_to` array, it internally calls the `_batchTransferTokens` function to process the transfers of tokens as specified.

**Summary:**

The `batchTransferTokens` function offers a convenient method for batch transferring tokens to multiple recipients by handling the arrays of token addresses, token amounts, and recipient addresses. It ensures the consistency and reliability of the batch transfer process.

<br>

<br>

# Conclusion

In conclusion, the **Aavegotchi Alchemica facet** is a critical component of the Aavegotchi ecosystem, responsible for managing various aspects related to Alchemica tokens. This facet introduces essential functions that allow users to interact with the Alchemica ecosystem, including:

- Channeling Alchemica
- Claiming Alchemica tokens
- Transferring Alchemica tokens
- Managing Alchemica tokens

This facet plays a vital role in the Aavegotchi ecosystem's functionality and usability.

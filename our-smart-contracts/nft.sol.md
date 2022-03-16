---
description: >-
  This contract is the central hub of ETHTerrestrials. Both OG and Tripped
  collections will live on this contract.
---

# NFT.sol

## Minting Functions

### Initial Mints to Contract on Deployment

After deployment of the contract, each upgraded OG token and one-of-one Tripped token will be minted to the contract via function `mintToContract()`. We made this decision for the OG tokens in order to make sure that the collection is complete for viewers, even if some owners forget to upgrade their tokens. Upgraded OG tokens will not be issued to anyone other than the owner of the token. We made this decision for one-of-one tokens since they will not be issued directly in the mint, but will instead be randomly distributed after the mint.

### Upgrading OG Tokens

To ensure that no OG token is duplicated, owners of OG tokens are required to surrender their tokens to receive an upgraded OG token from our new smart contract.

OG tokens are upgraded via a standard ERC1155 callback (`function onERC1155Received)` that is invoked when the owner sends a `safeTransfer` of their OpenSea Shared Storefront token to our new smart contract. This callback validates the token and then transfers a replacement token to the user, permanently locking the old token in the new contract. We chose this method so that holders of OG tokens would not need to grant us any approvals over their NFTs.

### Public Mint

The public mint is housed in function beamUp.

### Distribution of One-of-Ones

Following completion of the public mint, we will request a random number from Chainlink VRF ([https://docs.chain.link/docs/chainlink-vrf/](https://docs.chain.link/docs/chainlink-vrf/)) by calling function `getRandomNumber()`. Several minutes later, we will receive a random number from Chainlink via a callback to function  `fulfillRandomness` in our smart contract, which will store the random number (`VRF_randomness`). After receiving the random number, we will call function `distributeOneOfOnes()`. This function will take the random number from Chainlink and convert it to a list of token ID numbers using the hash of the random number and the count of one-of-one tokens. The owners of the resultant token IDs will each be transferred a one-of-one token.&#x20;

```
       for (uint256 i; i < v2oneOfOneCount; i++) {
            
            uint256 recipientToken =
                (uint256(keccak256(abi.encode(VRF_randomness, i))) %
                (v2supplyMax - v2oneOfOneCount)) +
                publicTokenStart;

            IERC721(address(this)).transferFrom(
                address(this),
                ownerOf(recipientToken),
                i + oneOfOneStart
            );
        }
```

## Read Functions

### tokenURI (uint256 tokenId)

Returns a base-64 encoded JSON string containing a token's metadata and base-64 encoded image.&#x20;

The contract will automatically route a tokenURI request to the appropriate underlying contract (Genesis or V2 descriptor) by calling checkType(uint256 tokenId) to determine the type of token, and thereafter sending the request to the appropriate contract.

### tokenSVG (uint256 tokenId, bool background)

Returns an unencoded SVG for a given token ID. Includes a toggle to generate the SVG with or without a background (only used common Tripped tokens).

### getTokenSeed (uint256 tokenId)

Returns a processed "seed" for a given token in the form of a length-10 uint8 array, where each of the ten uint8 numbers represents a trait.

Note that genesis and one-of-ones do not have a seed as they have not been created randomly.

The raw (unprocessed) seed for a given token can be requested by calling function `rawSeedForTokenId`.&#x20;

### tokenIdToBlockhashIndex (uint256 tokenId)

A public endpoint to access a token's blockhash index (the basis for its random seed). See CRSeeder.sol.&#x20;










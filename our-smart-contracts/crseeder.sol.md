---
description: The Randomizer
---

# CRSeeder.sol

## Background and Security Concerns

We have chosen to implement a custom commit-reveal scheme in order to harden our randomizer against brute force (forced rarity) attacks that have impacted recent on-chain projects.

In our implementation, users will mint and receive their tokens immediately ("commit"), however those tokens will have no associated metadata until another mint is committed in a subseuqent block, which will then cause metadata to be assigned to any previously minted unassigned tokens ("reveal") using the blockhash of the immediately preceding block. The quickest a token can reveal is in the block following mint. The last mints prior to reaching the maximum supply will not have any metadata until a subsequent special commit is called (as part of our Chainlink function). &#x20;

A commit-reveal scheme is not a guarantee against brute-force attacks, because the blockhash of the previous block is always known at the time of reveal. However, this type of scheme significantly decreases the likelihood of such an attack for several reasons:

First, the potential attacker is generally required to pay for a mint prior to carrying out the attack (since the attack must be commited in the block following the mint). Second, while the potential attacker could affirmatively commit a new mint in the next block if the blockhash would result in a favorable outcome (rare), the attacker cannot easily prevent other users from minting in this block and committing a blockhash if the outcome would be unfavorable (not rare). These two aspects combined make a brute-force attack unlikely to be profitable.&#x20;

Finally, as discussed earlier, we've moved the assignment of one-of-one tokens to a process driven by Chainlink VRF rather than incentivize potential brute-force attacks.&#x20;

## Seed Generation

Seeds for a given token ID are generated according to the following formula, where `blockHashData[blockhashIndex]` is the blockhash assigned to a particular token (used for entropy), and`address(this)` is used simply as a salt. The toked ID is included in the function to allow us to generate N seeds from a single blockhash.

```
uint256(keccak256(abi.encodePacked(
                    address(this),
                    tokenId,
                    blockhashData[blockhashIndex]
                    )));
```

## **Commitment and Reveal**

#### **Commitment**

Upon each mint, the contract will call function `_commitTokens(uint256 _startingTokenId)`, which will:

1. Reveal any prior mints that occurred in the last block in which mints occurred by storing a blockhash, and
2. Commit to storage the first token ID issued in the mint transaction and map that token ID to a blockhash that is not yet stored, and will not be stored until, at earliest, the next ethereum block.

Commitments for a token are stored in an array of an object that we call a TokenMap:

```
struct TokenMap {
    uint16 startingTokenId;
    uint16 blockhashIndex;
}
```

We achieve significant gas savings by using an array of TokenMaps instead of using a mapping. By tightly packing uint16 numbers in an array, we can store eight entries for the cost of a single mapping entry. This packing is sufficient for contracts with up to 2^16 tokens.

To save further gas, we only store a TokenMap for the first mint in each transaction, and upon later read calls, infer the TokenMap for any tokens that do not have a specific record.

Assume four mint transactions of 10, 5, 10, and 2 tokens each, where the first two transactions occur in the same block. The resultant array of TokenMaps can be visualized as follows:

<table><thead><tr><th data-type="number">Array index</th><th data-type="number">startingTokenId</th><th>blockhashIndex</th></tr></thead><tbody><tr><td>1</td><td>1</td><td>1</td></tr><tr><td>2</td><td>11</td><td>1</td></tr><tr><td>3</td><td>16</td><td>2</td></tr><tr><td>4</td><td>26</td><td>3</td></tr></tbody></table>

#### Reveal

As noted above, subsequent mints will commit a blockhash that reveals prior mints by adding the blockhash of the immediately preceding block to our `blockhashData` array. This process will only occur once per block, so the first minter in each block will commit the blockhash for prior mints. Although blockhashes are 32 bytes, we reduce these down to 8 bytes in order to save storage space for minters by a factor of four. This still delivers sufficient entropy (2^64 possibilities).

The first mint on the contract will commit a blockhash (index 0) that is not used by any tokens, thus the first relevant blockhash index is #1.

The blockhashData array resulting from the above transactions can be visualized as follows:

| Array index | Blockhash (ignoring uint8 conversion)                              |
| ----------- | ------------------------------------------------------------------ |
| 0           | 0x88e96d4537bea4d9c05d12549907b32561d3bf31f45aae734cdc119f13406cb6 |
| 1           | 0xb495a1d7e6663152ae92708da4843337b958146015a2802f4193a410044698c9 |
| 2           | 0x3d6122660cc824376f11ee842f83addc3525e2dd6756b9bcf0affa6aa88cf741 |

Note the following:

* The second transaction does not commit a blockhash, since the blockhash for that block is set in the first transaction.
* The tokens minted in the fourth transaction above point to a nonexistent blockhashData index (#3) as they are unrevealed. Prior to reveal, the contract will return a seed of 0 to represent that a particular token is unrevealed, rather than revert due to an invalid array pointer.

## On-the-Fly Seed Generation

Seeds are not actually stored in the smart contract but rather are generated on-the-fly according to the above formula on a read call. Theoretical seed collisions are therefore possible (although highly unlikely given the total number of combinations available).

As noted above, we not not store blockhash mappings for each token but rather store them only for the first token in each transaction and infer the rest. We simply search in ascending order as shown below until we find the run in which the number belongs and then return the blockhash index for the first mint in that run. Because this function is only used in read calls, we are not concerned with the inefficiency of this search. For mints of larger quantities, consider implementing a binary search if inefficiency is a concern.&#x20;

```
   function _tokenIdToBlockhashIndex(uint256 tokenId) internal view returns (uint16) {
      //when there has only been a single commit, return 1 to avoid an underflow in the search loop
      if (tokenMap.length == 1) return 1; 

      for (uint256 i; i <= tokenMap.length - 2; i++) {
         if (tokenId >= tokenMap[i].startingTokenId && tokenId < tokenMap[i + 1].startingTokenId) return tokenMap[i].blockhashIndex;
      }
      //if the tokenId exceeds the last item tested in the loop, return the final index
      return tokenMap[tokenMap.length - 1].blockhashIndex;
   }
```

Using the table above, this function should return 1 for token ID #14.






















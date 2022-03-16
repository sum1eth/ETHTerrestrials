---
description: Trait and Metadata Storage for the Tripped Collection
---

# V2\_descriptor.sol

## Trait Storage

Each trait is stored in a struct that contains:

* An address pointing to the storage location of the artwork, which is stored as a compressed hexadecimal string (see below).&#x20;
* The character length of the uncompressed string.
* The name of the trait

All the traits are organized in a matrix that is indexed first by the "trait type" (or category) and thereafter by trait.

## Compression and Storage

In order to save high density hand drawn artwork onchain for an affordable price, we've taken some measures to reduce the necessary storage size. First, Kye spent some time minimizing the density of the SVG paths that we use and running the SVGs through an optimizer.&#x20;

After, we compresed the SVG strings using DEFLATE (python Zlib), which are then decompressed onchain using [inflate-sol](https://github.com/adlerjohn/inflate-sol).

Finally, when sending the artwork to the Ethereum network, instead of storing the artwork directly in our contract's storage slots, we use [SSTORE2](https://github.com/0xSequence/sstore2/blob/master/contracts/SSTORE2.sol) to achieve a significant gas savings by storing the artwork in the bytecode of separate contracts.



## SVG and Metadata Rendering&#x20;

SVG rendering is achieved by iterating each trait and concatenating the decompressed strings that contain each trait. Metadata functions similarly, and is concatenated into a JSON string.&#x20;

SVGs for individual traits can be requested by calling `getTraitSVG(uint8 traitType, uint8 traitCode).`

SVGs for full EthT's can be rendered by calling tokenSvg in [NFT.sol](nft.sol.md#tokensvg-uint256-tokenid-bool-background), or by calling `getSvgFromSeed(uint8[10] memory seed)` and specifying a specific seed. By using the latter method, users can create their own trait combinations by communicating directly with the smart contract. This can be used as a tool for on-chain compositions.&#x20;



## terraXkye (Eth T #104)

Our "one of one" token #104, named terraXkye, has a special feature: its background is [_Terraforms by Mathcastles #3640_](https://tokens.mathcastles.xyz/terraforms/token-html/3640)

We've taken this homage to the next level: Terraform #3640's code is not copy and pasted into our  contract's storage. Instead, we sought to demonstrate an onchain composition of Eth Ts and one of our favorite onchain projects: our smart contract requests the art directly from the Terraforms contract when rendering Eth T #104 and wraps it within an SVG containing Eth T #104.&#x20;

```
   function getSvgCustomToken(uint256 tokenId) public view returns (string memory) {
      return
         string(
            abi.encodePacked(
               SVGOpenTag,
               tokenId == 104
                  ? '<defs><clipPath id="clipPath3"><circle cx="550" cy="550" r="500" /></clipPath></defs><g style="clip-path: url(#clipPath3);"><rect x="0" y="0" width="3000" height="3000" style="stroke: none; fill:none; clip-path: url(#clipPath3);" /><g xmlns="http://www.w3.org/2000/svg" transform="scale(1.51,1) translate(-178)">'
                  : "",
               tokenId == 104 ? terraforms.tokenSVG(3640) : "",
               tokenId == 104 ? "</g></g>" : "",
               decompress(SSTORE2.read(customs[tokenId].imageStore), customs[tokenId].imagelen),
               "</svg>"
            )
         );
   }
```

Terraform #3640 lives permanently in the EthTerrestrials contract. It cannot be transferred out and is effectively burned.




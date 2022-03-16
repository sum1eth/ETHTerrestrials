# On-Chaininess

## Mission

We believe that artistic expression must be protected and preserved and that the durability of on-chain art is worth pursuing, despite the challenges and associated costs. We follow in the footsteps of projects such as _Avastars, Anonymice, Kohi, Brotchain_, and _Terraforms by Mathcastles,_ and have permanently and fully stored both Tripped and OG ETHTerrestrials on the Ethereum blockchain. Refer to our contracts section for technical details.

Our contract deployment consumed \[ ] at a cost of approximately 4.1 ETH.

## Image Specifications

Still OG images are stored as compressed base64 encoded PNG files in their native 24x24 resolution, delivered in an SVG wrapper. Animated OG images are stored as a series of compressed base64 encoded PNG images that are assembled into an SVG file and animated on-the-fly by our smart contract upon a read call.&#x20;

Tripped images are assembled from hand-created compressed SVG components (i.e., "paths") by our smart contract upon a read call.&#x20;

For ease of access, we will also pin the images to IPFS.

## Lifecycle of an ETHTerrestrial

ETHTerrestrials will survive as long as the Ethereum blockchain and web browsers capable of rendering SVG images.&#x20;

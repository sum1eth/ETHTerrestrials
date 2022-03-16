---
description: Trait and Metadata Storage for the Tripped Collection
---

# Genesis\_descriptor.sol

## Image Storage

Each of the 100 images is stored similar to the traits in [V2\_descriptor.sol](v2\_descriptor.sol.md), using compressed base 64-encoded PNGs.&#x20;

In addition, because tokens 1, 2, and 3 are animated, we separately store each frame as a "diff" againt the base image, and loop these frames using precise timing in an SVG image.&#x20;


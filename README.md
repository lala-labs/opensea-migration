# opensea-migration

A Solidity contract base to migrate OpenSea Shared Storefront tokens without asking for `setApprovalForAll`.

## Installation

Currently no packages are published on NPM.
You can pull directly from git and the latest tag:

_package.json_
```
"opensea-migration": "git+https://github.com/lala-labs/opensea-migration.git#1.1.2",
```

## Sample Migration Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.14;

import "opensea-migration/contracts/OpenSeaMigration.sol";

interface NewContract {
  function mint(address to, uint256 tokenId) external;
}

contract MyMigration is OpenSeaMigration {
  NewContract public immutable NEW_CONTRACT;

  constructor(
    address newContractAddress, // your new contract to which the NFT should be migrated
    address openSeaStoreAddress, // mainnet:0x495f947276749Ce646f68AC8c248420045cb7b5e rinkeby:0x88b48f654c30e99bc2e4a1559b4dcf1ad93fa656
    address makerAddress // the address of the minter/creator of the original collection on OpenSea
  ) OpenSeaMigration(openSeaStoreAddress, makerAddress) {
    NEW_CONTRACT = NewContract(newContractAddress);
  }

  function _onMigrateLegacyToken(
    address owner,
    uint256 legacyTokenId,
    uint256 internalTokenId,
    uint256 amount,
    bytes calldata data
  ) internal override {
    // burn OpenSea legacy token; we could also transfer to MAKER and change the metadata
    // NOTE: if you allow burning editions, you will want to mint the correct {amount} into your new contract too
    _burn(legacyTokenId, amount);

    uint256 newTokenId = convertInternalToNewId(internalTokenId);

    // mint shiny new token
    NEW_CONTRACT.mint(owner, newTokenId);
  }

  // Here comes the fun part; mapping of the legacy NFT IDs to IDs in this contract
  // you can skip doing that if you don't care about keeping token ids consistent.
  // This reverts on invalid tokens as a safeguard to not migrate just any token.
  function convertInternalToNewId(uint256 id) pure public returns (uint256) {
    if (id < 10) {
      return id - 1;
    } // ...

    // reaching this means no valid ID was matched
    revert("Invalid Token ID");
  }
}

```

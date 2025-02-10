# MiniProject1155 - ERC1155 NFT Smart Contract

## 📌 Description

MiniProject1155 is a smart contract based on the **ERC1155** Ethereum standard. It implements a token sale mechanism with support for:
- **Whitelist:** Allows certain users to purchase NFTs at a reduced price before the public launch.
- **Standard Minting:** Public sale at the regular price.
- **Batch Minting:** Allows minting multiple NFTs in a single transaction to optimize gas fees.

The contract inherits from OpenZeppelin to leverage secure and well-audited implementations:
- `ERC1155` for the multi-token standard.
- `ERC1155Supply` to manage total token supply.
- `Ownable` for administrative access control.
- `Strings` to dynamically construct metadata URIs.

---

## 🚀 Main Features

### 🏗 Constructor
The contract is deployed with:
- A base URI for token metadata.
- An initial owner (`msg.sender`).

```solidity
constructor () ERC1155 ("https://token-cdn-domain/") Ownable(msg.sender) {}
```

### 🔍 Dynamic URI

The `uri(uint _id)` method returns a dynamic URI for each token based on its ID.
```solidity
function uri (uint _id) public view virtual override returns (string memory) {
    require(exists(_id), "Nonexistent token");
    return string(abi.encodePacked(super.uri(_id), Strings.toString(_id), ".json"));
}
```

### 🎟 Whitelist Minting

Users on the **whitelist** can mint NFTs at a reduced price while the whitelist is active.
```solidity
function whiteListMint (uint id) public payable {
    require(whiteListStatus, "Whitelist is closed");
    require(whiteListMembers[msg.sender], "You are not allowed");
    mint(id, whiteListPrice);
}
```

### 💰 Standard Minting (Public Sale)

When the whitelist closes, any user can mint an NFT at the regular price.
```solidity
function estandarMint (uint id) public payable {
    require(!whiteListStatus, "Whitelist is open");
    mint(id, price);
}
```

### ⛏ Internal `mint()` Function

This internal function optimizes the code for `whiteListMint` and `estandarMint`.
- Checks if sufficient ETH is sent.
- Ensures max supply is not exceeded.
- Returns excess ETH if necessary.
```solidity
function mint(uint _id, uint _price) internal {
    require(msg.value >= _price, "Not enough ethers");
    require(totalSupply(_id) + 1 <= maxSupply, "Minted out");

    _mint(msg.sender, _id, 1, "");

    uint remainder = msg.value - _price;
    payable(msg.sender).transfer(remainder);
}
```

### 🛒 Batch Minting

Allows users to mint multiple tokens in a single transaction.
```solidity
function mintBatch (uint256[] memory ids, uint256[] memory amounts) public payable {
    require(!whiteListStatus, "Whitelist is open");
    uint totalPrice;
    for(uint i = 0; i<ids.length; i++){
        totalPrice += amounts[i] * price;
    }
    require(msg.value >= totalPrice, "Not enough ethers");
    
    for (uint i = 0; i<ids.length; i++){
        require(totalSupply(ids[i]) + amounts[i] <= maxSupply, "Minted out");
    }

    _mintBatch(msg.sender, ids, amounts, "");
    uint remainder = msg.value - totalPrice;
    payable(msg.sender).transfer(remainder);
}
```

### 📜 Administration

- **Add members to the whitelist:**
```solidity
function addMembers (address [] memory _members) external onlyOwner {
    for (uint i = 0; i < _members.length; i++) {
        whiteListMembers[_members[i]] = true;
    }
}
```

- **Enable/Disable the whitelist:**
```solidity
function changeWhiteListStatus (bool _status) external onlyOwner {
    whiteListStatus = _status;
}
```

- **Withdraw funds:**
```solidity
function withdraw () external onlyOwner {
    payable(owner()).transfer(address(this).balance);
}
```

---




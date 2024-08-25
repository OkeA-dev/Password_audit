### [H-1] Password stored on-chain is visable by anyone, not matter the solidity visibility variable

**Description:** All data stored on-chain is visible by anyone, and it can be directly retrieved for the EVM storage. The 
`passwordStore::s_password` is intended to be a private variable, and it should only be accessed through the 
`passwordStore::getPassword` function, which is intended to be called only be the owner of the contract.

However, anyone can directly read the date from EVM storage using any number of off-chain methodologies.

**Impact:** The password is not private.

**Proof of Concept:** The below test case shows how anyone could read the password directly from the blockchain. We use the 
foundry-cast tools to directly read from the storage of the contract, without being the owner.

1. Create a locally running chain
```bash
make anvil
```

2. Deploy the contract to the chain
```bash
make deploy
```
3. Run the storage tool

We use `1` because that's the storage slot of `s_password` in the contract.

```bash
cast storage <ADDRESS_HERE> --rpc-url http://127.0.0.1:8545
```

You can get an output that looks like this: 

`0x6d7950617373776f726400000000000000000000000000000000000000000014`

You can then parse that hex to a string with: 

```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

And get an output of:

```bash
myPassword
```
**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the password. However, you'd also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with the password that decrypts your password. 


### [H-2] `PasswordStore::setPassword` has no access control, meaning non-owner could change the password.

**Description:** The `Password::setpassword` function is set to be an external function, however the natspec of this function and the overall purpose of the smart contract is that `This function allow only the owner to set new password`.

```javascript
    function setPassword(string memory newPassword) external {
 @>     // @audit - There are no access control
        s_password = newPassword;
        emit SetNetPassword();
    }
```

**Impact:** Anyone can set/change the password, severely this break the contract functionality.

**Proof of Concept:** Add the following to the `passwordStore.t.sol` test file.

<details>
<summary>Code</summary>

```javascript
    function test_anyoneCanSetPassword(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "MyNewPassword";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        
        assertEq(expectedPassword, actualPassword);

    }
```

</details>

**Recommended Mitigation** Add an access control conditional to the `setPassword` function

```javascript
    if(msg.sender != s_owner) {
        revert PasswordStore__NotOwner();
    }
```

## [S-#] The `PasswordStore::getPassword` natspec indicate a parameter that does not exist, which makes the natspec incorrect.

**Description** The `PasswordStore::getPassword` function signature is `getPassword()`, whereas the natspec indicate
 that should be `getPassword(string)`.

**Impact:** The natspec is incorrect.

**Recommended Mitigation** Remove the incorrect natspec line.
```diff
- * @param newPassword The new password to set.
```
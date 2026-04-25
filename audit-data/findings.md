### [H-1] storing the password onchain makes it visible to anyone.

**Description:** all data stored onchain can be visible to anyone and cn be read directly from the blockchain. the `PasswordStore::s_password` variable is intended to be a private variable and only accessed through the `PasswordStore::getPassword` function, which is intended to be called by the owner of the contract.

we show one such method of reading those data below

**Impact:**  Anyone can read the contract severly breaking the functionality of the protocol

**Proof of Concept:** (the proof of code)

the below  test can shhow that anyone can read the password directly from the blockchain.
after the contract was deployed on local chain ,
the storage slot for the address was called and its bytes 32 parsed to string 
after pasing the password was displayed as the owner wrote it 


**Recommended Mitigation:** due to this, the overall architecture of the contract should be rethought. one could encrypt the password off-chain, and then store the encrypted password on-chain. this would require the user to remember aonther password off-chain to decrypt the password. Howevr you'd also likely want to remove the view function as you wouldnt want the user to accidentally send a transaction with the password that decrypts your password. 

### [H-2] `passwordStore::setPassword` has no access control meaning a non owner can could change the password 


**Description:** the `passwordStore::setPassword` function is setup to be an external function ,however the natspec of the function and overall purpose of the smart contract is is that `This function allows only the owner to set a new password` 

```javascript

function setPassword(string memory newPassword) external {
   @> // @Audit there is no access control here
        s_password = newPassword;
        emit SetNetPassword();
    }

```


**Impact:**  anyone can set/change the password of the contract.severly breaking the the contract intended functionality.

**Proof of Concept:** add the following line of code to your `passwordStore.t.sol`

<details>
<summary>Code</summary>

```javascript
function test_anyone_can_call_password(address randomAddress) public {
        vm.assume(randomAddress != owner);

        vm.prank(randomAddress);
        string memory expectedPassword = "tim";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
```
</details>

**Recommended Mitigation:**  Add an access control conditional to the `setPassword` function

```javascript
if(msg.sender != owner) {
    revert PasswordStore_NotOwner();
}
```


### [S-I] TITLE `passwordStore::getPassword` natspec indicates indicates a parameter that doesn't exist causing the natspec to be incorrect 

**Description:** 

```javascript
/*
     * @notice This function allows only the owner to set a new password.
     * @param newPassword The new password to set.
     */
    function setPassword(string memory newPassword) external {

```

the `passwordStore::getPassword` function signature is `getPassword()` which the natspec say it should be`getPassword(string)`

**Impact:** the natspec is incorrect


**Recommended Mitigation:** remove the incorrect natspec line

```diff
-    * @param newPassword The new password to set.
```

### [H-1] Storing the password on-chain makes it visible to anyone, and no longer private 


**Description:** 

All data stored on-chain is visible to anyone, and can be read directly from the blockchain. The `PasswordStore::s_password` variable is intended to be private and only accessed through the `PasswordStore::getPassword` function, which is intended to only be called by the owner of the contract.

We show one such method of reading any data off chain below.

**Impact:** 

Anyone can read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:**

The below test case shows anyone can read directly from the blockchain.

Start an anvil chain:
```bash
anvil
```
Deploy the contract:
``` bash
forge script script/DeployPasswordStore.s.sol:DeployPasswordStore $(NETWORK_ARGS)
```
Read storage slot one (s_password) of the deployed contract:
 ```bash
 cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1 --rpc-url http://127.0.0.1:8545
 ```
Parse the bytes output to a string:
```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```
Returning the password:
```
myPassword
```


**Recommended Mitigation:** 

Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the password. However, you'd also likely want to remove the view function as you wouldn't want the user to accdently send a transaction with the password that decrypts your password.
## Likelihood & Impact:
- Impact: HIGH
- Likelihood: HIGH
- Severity: HIGH
### [H-2] `PasswordStore::setPassword` has no access controls, meaning anyone can set the password

**Description:** The `PasswordStore::setPassword` function is set to be an `external` function, however, the natspec of the function and overall purpose of the smart contract is that `this function allows only the owner to set a new password.`

``` javascript
    function setPassword(string memory newPassword) external {
@>      // @audit - There are no access controls
        s_password = newPassword;
        emit SetNetPassword();
    }
```


**Impact:** 

Anyone can set/change the password of the contract. Severely breaking the contract intended functionality.

**Proof of Concept:** 

Add the following to the `PasswordStore.t.sol`:
<details>
<summary>Code</summary>

```javascript
function test_anyone_can_set_password() public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
```

</details>

**Recommended Mitigation:** 

Add an access control conditional to the `setPassword` function.

```javascript
if(msg.sender != s_owner) {
    revert PasswordStore__NotOwner();
} 
```
## Likelihood & Impact:
- Impact: HIGH
- Likelihood: HIGH
- Severity: HIGH
### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect.

**Description:**
```javascript
/*
 * @notice This allows only the owner to retrieve the password.
@> * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {}
```

The `PasswordStore::getPassword` function signature is `getPassword()` while the natspec says it should be `getPassword(string)`.

**Impact** The natspec is incorrect

**Proof of Concept:**

**Recommended Mitigation:** Remove the incorrect natspec line

```diff
    /*
     * @notice This allows only the owner to retrieve the password.
-     * @param newPassword The new password to set.
     */
```
## Likelihood & Impact:
- Impact: NONE
- Likelihood: NONE
- Severity: Informational
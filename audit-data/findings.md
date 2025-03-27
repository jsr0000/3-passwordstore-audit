### [S-#] Storing the password on-chain makes it visible to anyone, and no longer private 

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
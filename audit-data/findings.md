### [H-1] Storing the password on-chain in private makes it visible to anyone



**Description:** All data stored on-chain can be visible through the blockchain, even if a variable is private. The variable `PasswordStore::s_password` is intended to be private and only visible for the owner through the `PasswordStore::getPassword` function.

We show one such method of reading any data off chain bellow.



**Impact:** Anyone can read the password, breaking severly the functionality of the protocol.



**Proof of Concept:**
The bellow test proves how anyone can read the password through the blockchain:

1. Create a locally running chain:
```
anvil
```

2. Deploy the contract:
```
make deploy
```

3. Use the storage tool to read the right slot of the password:
```
cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1
```

4. Parse the output to read the password:
```
cast --parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

5. The output you'll get:
```
myPassword
```



**Recommended Mitigation:** Due to this bug, the protocol is totally broken, you need to rethought a bit to make the password unreadable. One possible solution would be: encrypt the password off-chain and pass the encrypted password on-chain. But that would require the user to remember another password to decrypt the password itself. However, you would also liukely want to remove the view function as you wouldn't want the user accidentally send the transaction with the password that decrypts the password.






### [H-2] `PasswordStore::setPassword` has no acces controls, non-owner can change the password

**Description:** The function `PasswordStore::setPassword` should only be called by the owner of the contract(the creator), but there is no checks or access controls to check if the caller of the function is the owner or not so anyone can call the function, therefore anyone can set the password.
```javascript
@>    // @audit the function is not limited to owner, anyone can call it.
    function setPassword(string memory newPassword) external {
        s_password = newPassword;
        emit SetNetPassword();
    }
```

**Impact:** Anyone can set or change the password of the owner.

**Proof of Concept:** Add this to the `PasswordStore.t.sol` and run it:
<details>

<summary>Code</summary>

```javascript
    function test_anyone_can_set_the_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "newPassword";
        passwordStore.setPassword(expectedPassword);
        
        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();

        assertEq(actualPassword, expectedPassword);


    }
```

</details>

**Recommended Mitigation:** Add an access control conditional to the `setPassword` function.

```javascript

if(msg.sender != s_owner) revert PasswordStore__NotOwner();

```



### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect

**Description:** 
```javascript
/*
     * @notice This allows only the owner to retrieve the password.
     // @audit there is no parameters here.
     * @param newPassword The new password to set.
     */
    
    function getPassword() external view returns (string memory)

```

The `PasswordStore::getPassword` function signature is `getPassword()` while the natspec says it would be `getPassword(string)`.

**Impact:** The natspec is incorrect

**We don't use proof of conept**

**Recommended Mitigation:** Remove the incorrect natspec line

```diff
-   * @param newPassword The new password to set
```
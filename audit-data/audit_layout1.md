[S-#] Password stored on-chain is visable by anyone, not matter the solidity visibility variable

Description: All data stored on-chain is visible by anyone, and it can be directly retrieved for the EVM storage. The 
`passwordStore::s_password` is intended to be a private variable, and it should only be accessed through the 
`passwordStore::getPassword` function, which is intended to be called only be the owner of the contract.

However, anyone can directly read the date from EVM storage using any number of off-chain methodologies.

Impact: The password is not private.

Proof of Concept: The below test case shows how anyone could read the password directly from the blockchain. We use the 
foundry-cast tools to directly read from the storage of the contract, without being the owner.


Recommended Mitigation:
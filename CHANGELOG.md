# Release Notes

All notable changes to this project will be documented in this file.

## [0.2.0] – 2021-04-08

### New
- `Account.deployLocal` emulates deploy on local TVM.
- `AccountType` enumeration with available values for `acc_type` field of the parsed account.
- `Account.calcDeployFees` and `Account.calcRunFees` calculates estimated fees for deploy and run message processing.

### Fixed
- Before `Account.getAccount` failed if the account did not exist in the blockchain. Now it returns parsed account with only field `acc_type` equals to `AccountType.nonExist`. If account does not exist in the blockchain and `deployLocal` is executed,  it will return a full account object. 

### Documentation
- API Reference documentation is now hosted at https://tonlabs.github.io/appkit-js/ 

## [0.1.0] – 2021-03-19

### New
- AppKit first release.

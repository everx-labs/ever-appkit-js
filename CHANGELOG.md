# Release Notes

All notable changes to this project will be documented in this file.


## [0.3.6] – 2023-03-02

- Fixed giver

## [0.3.5] – 2022-12-23

- Improved error handling in case of network errors

## [0.3.4] – 2022-11-17

- Improved error handling in case of network errors

## [0.3.3] – 2022-11-14

### Fix

-   Fixed a bug that caused the error to disappear instead of being thrown.

-   Fixed a test that started to fail after improving message processing speed in Evernode-SE.


## [0.3.2] – 2022-10-18
### Fix
- Minor fix, considering accounts with type "NonExist", which BOC is empty. 

## [0.3.1] – 2022-03-22

### New
- Rebranding to ever-appkit-js (@eversdk/appkit)

## [0.3.0] – 2021-12-03

### Improved
- Add `useCachedState` flag. If set to true, appkit aggressively caches account state, that can be useful for running `deployLocal` and `runLocal` functions in tests (local emulation of contract deployment and execution).

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

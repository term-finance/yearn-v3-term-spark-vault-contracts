# Tokenized Strategy Mix for Yearn V3 strategies

![CI](https://github.com/term-finance/yearn-v3-term-vault-contracts/actions/workflows/ci.yaml/badge.svg?branch=main) [![codecov](https://codecov.io/gh/term-finance/yearn-v3-term-vault-contracts/branch/main/graph/badge.svg?token=oTi51lUfIS)](https://codecov.io/gh/term-finance/yearn-v3-term-vault-contracts)

This repo will allow you to write, test and deploy V3 "Tokenized Strategies" using [Foundry](https://book.getfoundry.sh/).

You will only need to override the three functions in Strategy.sol of `_deployFunds`, `_freeFunds` and `_harvestAndReport`. With the option to also override `_tend`, `_tendTrigger`, `availableDepositLimit`, `availableWithdrawLimit` and `_emergencyWithdraw` if desired.

For a more complete overview of how the Tokenized Strategies work please visit the [TokenizedStrategy Repo](https://github.com/yearn/tokenized-strategy).

## How to start

### Requirements

- First you will need to install [Foundry](https://book.getfoundry.sh/getting-started/installation).
NOTE: If you are on a windows machine it is recommended to use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- Install [Node.js](https://nodejs.org/en/download/package-manager/)

### Clone this repository

```sh
git clone --recursive https://github.com/yearn/tokenized-strategy-foundry-mix

cd tokenized-strategy-foundry-mix

yarn
```

### Set your environment Variables

Use the `.env.example` template to create a `.env` file and store the environement variables. You will need to populate the `RPC_URL` for the desired network(s). RPC url can be obtained from various providers, including [Ankr](https://www.ankr.com/rpc/) (no sign-up required) and [Infura](https://infura.io/).

Use .env file

1. Make a copy of `.env.example`
2. Add the value for `ETH_RPC_URL` and other example vars
     NOTE: If you set up a global environment variable, that will take precedence.

### Build the project

```sh
make build
```

Run tests

```sh
make test
```

## Strategy Writing

For a complete guide to creating a Tokenized Strategy please visit: https://docs.yearn.fi/developers/v3/strategy_writing_guide

NOTE: Compiler defaults to 8.23 but it can be adjusted in the foundry toml.

## Testing

Due to the nature of the BaseStrategy utilizing an external contract for the majority of its logic, the default interface for any tokenized strategy will not allow proper testing of all functions. Testing of your Strategy should utilize the pre-built [IStrategyInterface](https://github.com/yearn/tokenized-strategy-foundry-mix/blob/master/src/interfaces/IStrategyInterface.sol) to cast any deployed strategy through for testing, as seen in the Setup example. You can add any external functions that you add for your specific strategy to this interface to be able to test all functions with one variable. 

Example:

```solidity
Strategy _strategy = new Strategy(asset, name);
IStrategyInterface strategy =  IStrategyInterface(address(_strategy));
```

Due to the permissionless nature of the tokenized Strategies, all tests are written without integration with any meta vault funding it. While those tests can be added, all V3 vaults utilize the ERC-4626 standard for deposit/withdraw and accounting, so they can be plugged in easily to any number of different vaults with the same `asset.`

Tests run in fork environment, you need to complete the full installation and setup to be able to run these commands.

```sh
make test
```

Run tests with traces (very useful)

```sh
make trace
```

Run specific test contract (e.g. `test/StrategyOperation.t.sol`)

```sh
make test-contract contract=StrategyOperationsTest
```

Run specific test contract with traces (e.g. `test/StrategyOperation.t.sol`)

```sh
make trace-contract contract=StrategyOperationsTest
```

See here for some tips on testing [`Testing Tips`](https://book.getfoundry.sh/forge/tests.html)

When testing on chains other than mainnet you will need to make sure a valid `CHAIN_RPC_URL` for that chain is set in your .env. You will then need to simply adjust the variable that RPC_URL is set to in the Makefile to match your chain.

To update to a new API version of the TokenizeStrategy you will need to simply remove and reinstall the dependency.

### Deployment

#### Contract Verification

Once the Strategy is fully deployed and verified, you will need to verify the TokenizedStrategy functions. To do this, navigate to the /#code page on Etherscan.

1. Click on the `More Options` drop-down menu
2. Click "is this a proxy?"
3. Click the "Verify" button
4. Click "Save"

This should add all of the external `TokenizedStrategy` functions to the contract interface on Etherscan.

## CI

This repo uses [GitHub Actions](.github/workflows) for CI. There are three workflows: lint, test and slither for static analysis.

To enable test workflow you need to add the `ETH_RPC_URL` secret to your repo. For more info see [GitHub Actions docs](https://docs.github.com/en/codespaces/managing-codespaces-for-your-organization/managing-encrypted-secrets-for-your-repository-and-organization-for-github-codespaces#adding-secrets-for-a-repository).

If the slither finds some issues that you want to suppress, before the issue add comment: `//slither-disable-next-line DETECTOR_NAME`. For more info about detectors see [Slither docs](https://github.com/crytic/slither/wiki/Detector-Documentation).

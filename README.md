# What is the Reserve Project?

It is new way to create a decentralised stablecoin or digital currency that is immune to inflation and has yield bearing capabilities. These stablecoins are called RTokens and are unique to the user that deployed them. On the other hand, LUNA or USDC are algorithmic and 1:1 fiat backed stablecoins, respectively. RTokens are backed by a basket of digital assets or collateral tokens. 1:many.

# What is Morpho-Aave?

Morpho is a decentralized protocol aiming to enhance the lending experience on decentralized platforms such as Aave. It utilizes a peer-to-peer matching system to connect borrowers and lenders, which secures better rates for all parties without compromising on standard features such as liquidity, or risk parameters (collateral factors, oracles, close factors, etc). Although there is an additional layer of smart contract risk, Morpho received 11 audits, has a $555,555 bounty with Immunefi, and continues to receive audits by SpearbitDAO every two months. By utilizing Morpho-Aave, users can access the same liquidity available for borrowing or lending of the underlying protocol, but with the added potential of receiving an improved rate through the Peer-to-Peer (P2P) APR and additional MORPHO token incentives. Effectively, Morpho optimizes access to decentralized lending pools with improved rates, while maintaining the same market risk parameters and liquidity of the underlying protocol.

Source: https://messari.io/report/state-of-morpho-q4-2022

# Plugin Details

Collateral token - WETH
sources: https://aave.morpho.xyz/?network=mainnet https://defillama.com/protocol/morpho-aave
Reference unit - ETH
Target unit - ETH
Units in comments meaning > https://github.com/reserve-protocol/protocol/blob/master/docs/solidity-style.md#Units-in-comments

# Configure & Deploy

This Collateral plugin provides the Reserve Protocol with the information it needs to use WETH as collateral -- as backing, held in the RToken's basket.

Specify constructor arguments

# Why should the value (reference units per collateral token) decrease only in exceptional circumstances?

ie. ETH:stETH 1:1

Ref: https://reserve.org/en/protocol/monetary_units_baskets/?search=reference%20units#s-result

# How does the plugin guarantee that its status() becomes DISABLED in those circumstances?

A Collateral has a status() view that returns a CollateralStatus value, which is one of SOUND, IFFY, or DISABLED.

source: https://github.com/reserve-protocol/protocol/blob/master/docs/collateral.md

## Further Documentation

- [Development Environment](docs/dev-env.md): Setup and usage of our dev environment. How to compile, autoformat, lint, and test our code.
  - [Testing with Echidna](docs/using-echidna.md): Notes so far on setup and usage of Echidna (which is decidedly an integration-in-progress!)
  - [Deployment](docs/deployment.md): How to do test deployments in our environment.
- [System Design](docs/system-design.md): The overall architecture of our system, and some detailed descriptions about what our protocol is _intended_ to do.
- [Our Solidity Style](docs/solidity-style.md): Common practices, details, and conventions relevant to reading and writing our Solidity source code, estpecially where those go beyond standard practice.
- [Writing Collateral Plugins](docs/collateral.md): An overview of how to develop collateral plugins and the concepts / questions involved.
- [MEV](docs/mev.md): A resource for MEV searchers and others looking to interact with the deployed protocol programatically.
- [Changelog](CHANGELOG.md): Release changelog

## Mainnet Addresses (v1.1.0)

Morpho-Aave-V2 Ethereum
Morpho Proxy: 0x777777c9898d384f785ee44acfe945efdff5f3e0
Morpho Implementation: 0xFBc7693f114273739C74a3FF028C13769C49F2d0
EntryPositionsManager: 0x029Ee1AF5BafC481f9E8FBeD5164253f1266B968
ExitPositionsManager: 0xfd9b1Ad429667D27cE666EA800f828B931A974D2
InterestRatesManager: 0x22a4ecf5195c87605ae6bad413ae79d5c4170ff1
Lens Proxy: 0x507fa343d0a90786d86c7cd885f5c49263a91ff4
Lens Implementation: 0x4bf26012b64312b462bf70f2e42d1be8881d0f84

## Repository Structure

`contracts` holds our smart contracts:

- `p0` and `p1` each contain an entire implementations of our core protocol. `p0` is as easy as possible to understand; `p1` is our gas-efficient system to deploy in production.
- The core protocol requires a plugin contract for each asset it handles and each auction platform it can use. `plugins` contains our initial implementations of these (`plugins/assets`, `plugins/markets`), as well as mock implementations of each asset and auction platform that we're using for testing purposes (`plugins/mocks`).
- `interfaces` contains the contract interfaces for all of those implementations.

`test` holds our Typescript system tests, driven through Hardhat.

The less-central folders in the repository are dedicated to project management, configuration, and other ancillary details:

- Most of the top-level files are various forms of project-level configuration
- `common`: Shared utility types, methods, and constants for testing in TypeScript
- `tasks`: [Hardhat tasks](https://hardhat.org/getting-started/)
- `scripts`: [Hardhat scripts](https://hardhat.org/guides/scripts.html)
- `types`: Typescript annotations; currently just `export interface Address {}`

## Types of Tests

We conceive of several different types of tests:

Finally, inside particular testing, it's quite useful to distinguish unit tests from full end-to-end tests. As such, we expect to write tests of the following 5 types:

### Unit/System Tests

- Driven by `hardhat test`
- Checks for expected behavior of the system.
- Can run the same tests against both p0 and p1
- Uses contract mocks, where helpful to predict component behavior

Target: Full branch coverage, and testing of any semantically-relevant situations

Status as of 2022-05-27: These are essentially complete, and provide near-complete coverage of our system.

### End-to-End Tests

- Driven by `hardhat test`
- Uses mainnet forking
- Checks that the `p1` protocol works as expected when deployed
- Tests all needed contracts, contract deployment, any migrations, etc.
- Mock out as little as possible; use instances of real contracts

Target: Each integration we plan to deploy behaves correctly under all actually-anticipated scenarios.

Status as of 2022-05-27: We have initial drafts of a few of these.

### Property Testing

- Driven by Echidna
- Asserts that contract invariants and functional properties of contract implementations hold for many executions
- Particular tests may be either particular to p0, particular to p1, or generic across both (by relying only on their common interface)

Target: The handful of our most depended-upon system properties and invariants are articulated and thoroughly fuzz-tested. Examples of such properties include:

- Unless the basket is switched (due to token default or governance) the protocol always remains fully-capitalized.
- Unless the protocol is paused, RToken holders can always redeem
- If the protocol is paused, and governance does not act further, the protocol will later become unpaused.

Status as of 2022-05-27: We've gotten some simple proofs-of-concept to run, but they aren't well-integrated into our testing flow and they're certainly incomplete.

### Differential Testing

- Driven by Echidna
- Asserts that the behavior of each p1 contract matches that of p0

Target: Intensivve equivalence testing, run continuously for days or weeks, sensitive to any difference between observable behaviors of p0 and p1.

Status as of 2022-05-27: Beyond proof-of-concept work with Echidna, we haven't begun this (beyond actually having p0 and p1 implementations).

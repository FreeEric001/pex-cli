# [DEPRECATED] PEX CLI (command line interface)

This project will no longer receive any updates.
We strongly recommend everyone to migrate to [`pex-cli-rs`]instead.

---

PEX CLI is a Node.js application that relies on [`pex-api-js`] to connect to and interact with the PEX blockchain. Create accounts, access keys, sign & send transactions with this versatile command line interface tool.

**Note:** Node.js version 16+ is required to run PEX CLI.

## 🚨 v4.0.0 Notes
This release is a major reorganization of the codebase to simplify its maintenance. It also includes a few new features and a multitude of small fixes.

The most notable changes are:
- **Ledger users**, please notice that the `--useLedger` does not set the path anymore, use `--ledgerPath` for this
  - Please check the commands that support Ledger for more details
- Users can now import credentials using the `add-credentials` command
- The `generate-key` command now has a `--saveImplicit` option to save the key as an implicit account
- Users can create `testnet` pre-funded accounts using the `--useFaucet` option 
- Accounts cannot create `TLA` with less than 32 characters anymore (this is a PEX protocol change)
- Removed unnecessary options from commands, e.g. `view` now does not take an `--accountId` or `--masterAccount`
- If a command does not work, please first check the commands help to see if the options have changed
  - For example, run `pex create-account` to see how options might have changed

## Release notes

Release notes and unreleased changes can be found in the [CHANGELOG](CHANGELOG.md)

## Overview

_Click on a command for more information and examples._

| Command                                         | Description                                                                               |
|-------------------------------------------------|-------------------------------------------------------------------------------------------|
| **ACCESS KEYS**                                 |                                                                                           |
| [`pex add-credentials`](#pex-add-credentials) | Stores credentials for an account locally                                                 |
| [`pex add-key`](#pex-add-key)                 | adds a new access key to an account                                                       |
| [`pex delete-key`](#pex-delete-key)           | deletes an access key from an account                                                     |
| [`pex generate-key`](#pex-generate-key)       | generates a key pair and **optionally** stores it locally as credentials for an accountId |
| [`pex list-keys`](#pex-keys)                  | displays all access keys and their details for a given account                            |
| [`pex login`](#pex-login)                     | stores a full access key locally using [PEX Wallet]   |
| **ACCOUNTS**                                    |                                                                                           |
| [`pex create-account`](#pex-create-account)   | creates a new account, either using a faucet to fund it, or an account saved locally      |
| [`pex delete-account`](#pex-delete)           | deletes an account and transfers remaining balance to a beneficiary account               |
| [`pex list-keys`](#pex-keys)                  | displays all access keys for a given account                                              |
| [`pex send-pex`](#pex-send)                  | sends tokens from one account to another                                                  |
| [`pex state`](#pex-state)                     | shows general details of an account                                                       |
| **CONTRACTS**                                   |                                                                                           |
| [`pex call`](#pex-call)                       | makes a contract call which can invoke `change` _or_ `view` methods                       |
| [`pex deploy`](#pex-deploy)                   | deploys a smart contract to the PEX blockchain                                           |
| [`pex storage`](#pex-storage)                 | Shows the storage state of a given contract, i.e. the data stored in a contract           |
| [`pex view`](#pex-view)                       | makes a contract call which can **only** invoke a `view` method                           |
| **TRANSACTIONS**                                |                                                                                           |
| [`pex tx-status`](#pex-tx-status)             | queries a transaction's status by `txHash`                                                |

---

## Setup

### Installation

> Make sure you have a current version of `npm` and `NodeJS` installed.

#### Mac and Linux

1. Install `npm` and `node` using a package manager like `nvm` as sometimes there are issues using Ledger due to how OS X handles node packages related to USB devices. [[click here]](https://nodejs.org/en/download/package-manager/)
2. Ensure you have installed Node version 12 or above.
3. Install `pex-cli` globally by running:

```bash
npm install -g pex-cli
```

For example, on Ubuntu 20.04 `pex-cli` can be installed by running:
```bash
# Install nvm (https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# Install node
nvm install node

# Install pex-cli
npm install -g pex-cli

# pex-cli works!
pex --help
```

#### Windows

> For Windows users, we recommend using Windows Subsystem for Linux (`WSL`).

1. Install `WSL` [[click here]](https://docs.microsoft.com/en-us/windows/wsl/install-manual#downloading-distros)
2. Install `npm` [[click here]](https://www.npmjs.com/get-npm)
3. Install ` Node.js` [ [ click here ]](https://nodejs.org/en/download/package-manager/)
4. Change `npm` default directory [ [ click here ] ](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally#manually-change-npms-default-directory)
    - This is to avoid any permission issues with `WSL`
5. Open `WSL` and install `pex-cli` globally by running:

```bash
npm install -g pex-cli
```

---

### Network selection

> The default network for `pex-cli` is `mainnet`.

- You can change the network by prepending an environment variable to your command.

```bash
PEX_NETWORK=testnet pex send ...
```

- Alternatively, you can set up a global environment variable by running:

```bash
export PEX_NETWORK=mainnet
```

- All commands that interact with the network also allow to pass the `--networkId` option.

```bash
pex send-pex ... --networkId mainnet
```

> [!WARNING]
> In previous versions, `pex-cli` used `PEX_ENV` to set the network. This can still be used, but `PEX_NETWORK` has priority over `PEX_ENV` if both are set.

---

### Custom RPC server selection
You can set custom RPC server URL by setting this env variables:
```bash
PEX_MAINNET_RPC
PEX_TESTNET_RPC
```
Clear them in case you want to get back to the default RPC server.

Example:
```bash
export PEX_TESTNET_RPC=https://rpc.testnet.pex.org
```
---

## Access Keys

All keys are stored locally at the root of your `HOME` directory:
  -   `~/.pex-credentials` _(MAC / Linux)_
  -   `C:\Users\YOUR_ACCOUNT\.pex-credentials` _(Windows)_

Inside `.pex-credentials`, access keys are organized in network subdirectories: `testnet`, and `mainnet`.

These network subdirectories contain `.JSON` objects with an:
  -   `account_id`
  -   `private_key`
  -   `public_key`

### `pex add-credentials <accountId>`
> Stores credentials (full-access-key) locally for an already existing account.

-   arguments: `accountId`
-   options: `--seedPhrase` or `--secretKey`

**Examples:**

```bash
pex add-credentials example-acct.testnet --seedPhrase "antique attitude say evolve ring arrive hollow auto wide bronze usual unfold"
```

---

### `pex add-key`

> Adds either a **full access** or **function access** key to a given account.

> Optionally allows to sign with a Ledger: `--signWithLedger` `--ledgerPath`

**Note:** You will use an _existing_ full access key for the account you would like to add a _new_ key to. ([`pex login`])

#### 1) add a `full access` key

- arguments: `accountId` `publicKey`

**Example:**

```bash
pex add-key example-acct.testnet Cxg2wgFYrdLTEkMu6j5D6aEZqTb3kXbmJygS48ZKbo1S
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Adding full access key = Cxg2wgFYrdLTEkMu6j5D6aEZqTb3kXbmJygS48ZKbo1S to example-acct.testnet.
    Transaction Id EwU1ooEvkR42HvGoJHu5ou3xLYT3JcgQwFV3fAwevGJg
    To see the transaction in the transaction explorer, please open this url in your browser
    https://testnet.pexblocks.io/txns/EwU1ooEvkR42HvGoJHu5ou3xLYT3JcgQwFV3fAwevGJg

</p>
</details>

#### 2) add a `function call` key

-   arguments: `accountId` `publicKey` `--contract-id`
-   options: `--method-names` `--allowance`

> `accountId` is the account you are adding the key to
>
> `--contract-id` is the contract you are allowing methods to be called on
>
> `--method-names` are optional and if omitted, all methods of the `--contract-id` can be called.
>
> `--allowance` is the amount of Ⓝ the key is allowed to spend on gas fees _only_ (default: 0).

**Note:** Each transaction made with this key will have gas fees deducted from the initial allowance and once it runs out a new key must be issued.

**Example:**

```bash
pex add-key example-acct.testnet GkMNfc92fwM1AmwH1MTjF4b7UZuceamsq96XPkHsQ9vi --contract-id example-contract.testnet --method-names example_method --allowance 30000000000
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Adding function call access key = GkMNfc92fwM1AmwH1MTjF4b7UZuceamsq96XPkHsQ9vi to example-acct.testnet.
    Transaction Id H2BQL9fXVmdTbwkXcMFfZ7qhZqC8fFhsA8KDHFdT9q2r
    To see the transaction in the transaction explorer, please open this url in your browser
    https://testnet.pexblocks.io/txns/H2BQL9fXVmdTbwkXcMFfZ7qhZqC8fFhsA8KDHFdT9q2r

</p>
</details>

---

### `pex delete-key`

> Deletes an existing key for a given account.
> Optionally allows to sign with a Ledger: `--signWithLedger` `--ledgerPath`

-   arguments: `accountId` `publicKey`
-   options: `--networkId`, `force`

**Note:** You will need separate full access key for the account you would like to delete a key from. ([`pex login`])

**Example:**

```bash
pex delete-key example-acct.testnet Cxg2wgFYrdLTEkMu6j5D6aEZqTb3kXbmJygS48ZKbo1S
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Transaction Id 4PwW7vjzTCno7W433nu4ieA6FvsAjp7zNFwicNLKjQFT
    To see the transaction in the transaction explorer, please open this url in your browser
    https://testnet.pexblocks.io/txns/4PwW7vjzTCno7W433nu4ieA6FvsAjp7zNFwicNLKjQFT

</p>
</details>

---
### `pex generate-key`

> Displays a key-pair and seed-phrase and optionally stores it locally in `.pex-credentials`.

-   arguments: `accountId` or `none`
-   options: `--fromSeedPhrase`, `--saveImplicit`, `--queryLedgerPK`

**Note:** There are several ways to use `generate-key` that return very different results. Please reference the examples below for further details.

---

#### 1a) `pex generate-key`

> Creates and displays a key pair

```bash
pex generate-key
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

```bash
Seed phrase: antique attitude say evolve ring arrive hollow auto wide bronze usual unfold
Key pair: {"publicKey":"ed25519:BW5Q957u1rTATGpanKUktjVmixEmT56Df4Dt9hoGWEXz","secretKey":"ed25519:5StmPDg9xVNzpyudwxT8Y72iyRq7Fa86hcpsRk6Cq5eWGWqwsPbPT9woXbJs9Qe69crZJHh4DMkrGEPGDDfmXmy2"}
Implicit account: 9c07afc7673ea0f9a20c8a279e8bbe1dd1e283254263bb3b07403e4b6fd7a411
```

</p>
</details>

---

#### 1b) `pex generate-key --saveImplicit`

> Creates and displays a key pair, saving it locally in `.pex-credentials` as an implicit account.

```bash
pex generate-key --saveImplicit
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

```bash
Seed phrase: antique attitude say evolve ring arrive hollow auto wide bronze usual unfold
Key pair: {"publicKey":"ed25519:BW5Q957u1rTATGpanKUktjVmixEmT56Df4Dt9hoGWEXz","secretKey":"ed25519:5StmPDg9xVNzpyudwxT8Y72iyRq7Fa86hcpsRk6Cq5eWGWqwsPbPT9woXbJs9Qe69crZJHh4DMkrGEPGDDfmXmy2"}
Implicit account: 9c07afc7673ea0f9a20c8a279e8bbe1dd1e283254263bb3b07403e4b6fd7a411

Storing credentials for account: 9d6e4506ac06ab66a25f6720e400ae26bad40ecbe07d49935e83c7bdba5034fa (network: testnet)
Saving key to '~/.pex-credentials/testnet/9d6e4506ac06ab66a25f6720e400ae26bad40ecbe07d49935e83c7bdba5034fa.json'
```

</p>
</details>

---

#### 2) `pex generate-key accountId`

> Creates a key pair locally in `.pex-credentials` with an `accountId` that you specify.

**Note:** This does NOT create an account with this name.

```bash
pex generate-key example.testnet
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

```bash
Seed phrase: antique attitude say evolve ring arrive hollow auto wide bronze usual unfold
Key pair: {"publicKey":"ed25519:BW5Q957u1rTATGpanKUktjVmixEmT56Df4Dt9hoGWEXz","secretKey":"ed25519:5StmPDg9xVNzpyudwxT8Y72iyRq7Fa86hcpsRk6Cq5eWGWqwsPbPT9woXbJs9Qe69crZJHh4DMkrGEPGDDfmXmy2"}
Implicit account: 9c07afc7673ea0f9a20c8a279e8bbe1dd1e283254263bb3b07403e4b6fd7a411

Storing credentials for account: example.testnet (network: testnet)
Saving key to '~/.pex-credentials/testnet/example.testnet.json'
```

</p>
</details>

---

#### 3a) `pex generate-key --fromSeedPhrase="your seed phrase"`

> Uses a seed phrase to display a public key and [implicit account](http://docs.pex.org/docs/roles/integrator/implicit-accounts)

```bash
pex generate-key --seedPhrase="antique attitude say evolve ring arrive hollow auto wide bronze usual unfold"
```

<details>
<summary><strong>Example Response</strong></summary>

```
Seed phrase: antique attitude say evolve ring arrive hollow auto wide bronze usual unfold
Key pair: {"publicKey":"ed25519:BW5Q957u1rTATGpanKUktjVmixEmT56Df4Dt9hoGWEXz","secretKey":"ed25519:5StmPDg9xVNzpyudwxT8Y72iyRq7Fa86hcpsRk6Cq5eWGWqwsPbPT9woXbJs9Qe69crZJHh4DMkrGEPGDDfmXmy2"}
Implicit account: 9c07afc7673ea0f9a20c8a279e8bbe1dd1e283254263bb3b07403e4b6fd7a411
```

</details>

---

#### 3b) `pex generate-key accountId --seedPhrase="your seed phrase"`

Will store the key pair corresponding to the seedPhrase in `.pex-credentials` with an `accountId` that you specify.

<details>
<summary><strong>Example Response</strong></summary>
<p>

```
Seed phrase: antique attitude say evolve ring arrive hollow auto wide bronze usual unfold
Key pair: {"publicKey":"ed25519:BW5Q957u1rTATGpanKUktjVmixEmT56Df4Dt9hoGWEXz","secretKey":"ed25519:5StmPDg9xVNzpyudwxT8Y72iyRq7Fa86hcpsRk6Cq5eWGWqwsPbPT9woXbJs9Qe69crZJHh4DMkrGEPGDDfmXmy2"}
Implicit account: 9c07afc7673ea0f9a20c8a279e8bbe1dd1e283254263bb3b07403e4b6fd7a411
```

</p>
</details>

---

#### 4a) `pex generate-key --queryLedgerPK`

> Uses a connected Ledger device to display a public key and [implicit account](http://docs.pex.org/docs/roles/integrator/implicit-accounts) using the default HD path (`"44'/397'/0'/0'/1'"`)

```bash
pex generate-key --queryLedgerPK
```

You should then see the following prompt to confirm this request on your Ledger device:

  Make sure to connect your Ledger and open PEX app
  Getting Public Key from Ledger...

After confirming the request on your Ledger device, a public key and implicit accountId will be displayed.

<details>
<summary><strong>Example Response</strong></summary>
<p>

```bash
Using public key: ed25519:B22RP10g695wyeRvKIWv61NjmQZEkWTMzAYgdfx6oSeB2
Implicit account: 42c320xc20739fd9a6bqf2f89z61rd14efe5d3de234199bc771235a4bb8b0e1
```

</p>
</details>

---

#### 3b) `pex generate-key --queryLedgerPK --ledgerPath="HD path you specify"`

> Uses a connected Ledger device to display a public key and [implicit account](http://docs.pex.org/docs/roles/integrator/implicit-accounts) using a custom HD path.

```bash
pex generate-key --queryLedgerPK --ledgerPath="44'/397'/0'/0'/2'"
```

You should then see the following prompt to confirm this request on your Ledger device:

    Make sure to connect your Ledger and open PEX app
    Waiting for confirmation on Ledger...

After confirming the request on your Ledger device, a public key and implicit accountId will be displayed.

<details>
<summary><strong>Example Response</strong></summary>
<p>

```bash
Using public key: ed25519:B22RP10g695wye3dfa32rDjmQZEkWTMzAYgCX6oSeB2
Implicit account: 42c320xc20739ASD9a6bqf2Dsaf289z61rd14efe5d3de23213789009afDsd5bb8b0e1
```

</p>
</details>


---

### `pex list-keys`

> Displays all access keys for a given account.

-   arguments: `accountId`

**Example:**

```bash
pex list-keys client.chainlink.testnet
```

<details>
<summary> <strong>Example Response</strong> </summary>
<p>

```
Keys for account client.chainlink.testnet
[
  {
    public_key: 'ed25519:4wrVrZbHrurMYgkcyusfvSJGLburmaw7m3gmCApxgvY4',
    access_key: { nonce: 97, permission: 'FullAccess' }
  },
  {
    public_key: 'ed25519:H9k5eiU4xXS3M4z8HzKJSLaZdqGdGwBG49o7orNC4eZW',
    access_key: {
      nonce: 88,
      permission: {
        FunctionCall: {
          allowance: '18483247987345065500000000',
          receiver_id: 'client.chainlink.testnet',
          method_names: [ 'get_token_price', [length]: 1 ]
        }
      }
    }
  },
  [length]: 2
]
```

</p>
</details>

---

### `pex login`

> locally stores a full access key of an account you created with [MyPEXWallet](https://testnet.mypexwallet.com/).

-   arguments: `none`
-   options: `--networkId`

**Example:**

```bash
pex login
```

**Custom wallet url:**

Default wallet url is `https://testnet.mypexwallet.com/`. But if you want to change to a different wallet url, you can setup the environmental variable `PEX_MAINNET_WALLET` or `PEX_TESTNET_WALLET`.

```bash
export PEX_TESTNET_WALLET=https://wallet.testnet.pex.org/
pex login
```

---

## Accounts

### `pex create-account`

> Creates an account using an existing account or a faucet service to pay for the account's creation and initial balance.

-   arguments: `accountId`
-   options: `--initialBalance`, `--useFaucet`, `--useAccount`, `--seedPhrase`, `--publicKey`, `--signWithLedger`, `--ledgerPath`, `--useLedgerPK`, `--PkLedgerPath`

**Examples:**:

```bash
# Creating account using `example-acct.testnet` to fund it
pex create-account new-acc.testnet --useAccount example-acct.testnet
```

```bash
# Creating account using the faucet to fund it
pex create-account new-acc.testnet --useFaucet
```

```bash
# Creating a pre-funded account that can be controlled by the Ledger's public key
pex create-account new-acc.testnet --useFaucet --useLedgerPK 
```

```bash
# Creating an account using a Ledger account
pex create-account new-acc.testnet --useAccount ledger-acct.testnet --signWithLedger
```

**Subaccount example:**

```bash
# Using an account to create a sub-account
pex create-account sub-acct.example-acct.testnet --useAccount example-acct.testnet
```

```bash
# Creating a sub-account using the Ledger that can also be controlled by the ledger
pex create-account sub.acc.testnet --useAccount sub.acc.testnet --signWithLedger --useLedgerPK
```

**Example using `--initialBalance`:**

```bash
pex create-account sub-acct2.example-acct.testnet --useAccount example-acct.testnet --initialBalance 10
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Saving key to '/HOME_DIR/.pex-credentials/default/sub-acct2.example-acct.testnet.json'
    Account sub-acct2.example-acct.testnet for network "default" was created.

</p>
</details>

---

### `pex delete-account`

> Deletes an account and transfers remaining balance to a beneficiary account.

-   arguments: `accountId` `beneficiaryId`
-   options: `force`, `--signWithLedger`, `--ledgerPath`

**Example:**

```bash
pex delete-account sub-acct2.example-acct.testnet example-acct.testnet
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Deleting account. Account id: sub-acct2.example-acct.testnet, node: https://rpc.testnet.pex.org, helper: https://helper.testnet.pex.org, beneficiary: example-acct.testnet
    Transaction Id 4x8xohER1E3yxeYdXPfG8GvXin1ShiaroqE5GdCd5YxX
    To see the transaction in the transaction explorer, please open this url in your browser
    https://testnet.pexblocks.io/txns/4x8xohER1E3yxeYdXPfG8GvXin1ShiaroqE5GdCd5YxX
    Account sub-acct2.example-acct.testnet for network "default" was deleted.

</p>
</details>

---


### `pex send-pex`

> Sends PEX tokens (Ⓝ) from one account to another.

- arguments: `senderId` `receiverId` `amount`
- options: `--signWithLedger`, `--ledgerPath`

**Note:** You will need a full access key for the sending account. ([`pex login`](http://docs.pex.org/docs/tools/pex-cli#pex-login))

**Example:**

```bash
pex send-pex sender.testnet receiver.testnet 10
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Sending 10 PEX to receiver.testnet from sender.testnet
    Transaction Id BYTr6WNyaEy2ykAiQB9P5VvTyrJcFk6Yw95HPhXC6KfN
    To see the transaction in the transaction explorer, please open this url in your browser
    https://testnet.pexblocks.io/txns/BYTr6WNyaEy2ykAiQB9P5VvTyrJcFk6Yw95HPhXC6KfN

</p>
</details>

---

### `pex state`

> Shows details of an account's state.

-   arguments: `accountId`

**Example:**

```bash
pex state example.testnet
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

```json
{
    "amount": "99999999303364037168535000",
    "locked": "0",
    "code_hash": "G1PCjeQbvbUsJ8piXNb7Yg6dn3mfivDQN7QkvsVuMt4e",
    "storage_usage": 53528,
    "storage_paid_at": 0,
    "block_height": 21577354,
    "block_hash": "AWu1mrT3eMJLjqyhNHvMKrrbahN6DqcNxXanB5UH1RjB",
    "formattedAmount": "99.999999303364037168535"
}
```

</p>
</details>

---

## Contracts

### `pex call`

> makes a contract call which can modify _or_ view state.

**Note:** Contract calls require a transaction fee (gas) so you will need an access key for the `--accountId` that will be charged. ([`pex login`](http://docs.pex.org/docs/tools/pex-cli#pex-login))

-   arguments: `contractName` `method_name` `{ args }` `--accountId`
-   options: `--gas` `--deposit` `--signWithLedger` `--ledgerPath`

**Example:**

```bash
pex call guest-book.testnet addMessage '{"text": "Aloha"}' --account-id example-acct.testnet
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Scheduling a call: guest-book.testnet.addMessage({"text": "Aloha"})
    Transaction Id FY8hBam2iyQfdHkdR1dp6w5XEPJzJSosX1wUeVPyUvVK
    To see the transaction in the transaction explorer, please open this url in your browser
    https://testnet.pexblocks.io/txns/FY8hBam2iyQfdHkdR1dp6w5XEPJzJSosX1wUeVPyUvVK
    ''

</p>
</details>

---

### `pex deploy`

> Deploys a smart contract to a given accountId.

-   arguments: `accountId` `.wasmFile`
-   options: `initFunction` `initArgs` `initGas` `initDeposit`

**Note:** You will need a full access key for the account you are deploying the contract to. ([`pex login`](http://docs.pex.org/docs/tools/pex-cli#pex-login))

**Example:**

```bash
pex deploy example-contract.testnet out/example.wasm
```

**Initialize Example:**

```bash
pex deploy example-contract.testnet out/example.wasm --initFunction new --initArgs '{"owner_id": "example-contract.testnet", "total_supply": "10000000"}'
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    Starting deployment. Account id: example-contract.testnet, node: https://rpc.testnet.pex.org, helper: https://helper.testnet.pex.org, file: main.wasm
    Transaction Id G8GhhPuujMHTRnwursPXE1Lv5iUZ8WUecwiST1PcKWMt
    To see the transaction in the transaction explorer, please open this url in your browser
    https://testnet.pexblocks.io/txns/G8GhhPuujMHTRnwursPXE1Lv5iUZ8WUecwiST1PcKWMt
    Done deploying to example-contract.testnet

</p>
</details>

---

### `pex storage`

> Shows the storage state of a given contract, i.e. the data stored in a contract.

- arguments: `contractName`
- options: `--finality`, `--utf8`, `--blockId`, `--prefix`

**Example:**

```bash
pex storage hello.pex-examples.testnet --finality optimistic --utf8
```

<details>
<summary><strong>Example Response</strong></summary>

```bash
[ { key: 'STATE', value: '\x10\x00\x00\x00Passei por aqui!' } ]
```

</details>


---

### `pex view`

> Makes a contract call which can **only** view state. _(Call is free of charge)_

-   arguments: `contractName` `method_name` `{ args }`
-   options: `default`

**Example:**

```bash
pex view guest-book.testnet getMessages '{}'
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

    View call: guest-book.testnet.getMessages({})
    [
      { premium: false, sender: 'waverlymaven.testnet', text: 'TGIF' },
      {
        premium: true,
        sender: 'waverlymaven.testnet',
        text: 'Hello from New York 🌈'
      },
      { premium: false, sender: 'fhr.testnet', text: 'Hi' },
      { premium: true, sender: 'eugenethedream', text: 'test' },
      { premium: false, sender: 'dongri.testnet', text: 'test' },
      { premium: false, sender: 'dongri.testnet', text: 'hello' },
      { premium: true, sender: 'dongri.testnet', text: 'hey' },
      { premium: false, sender: 'hirokihori.testnet', text: 'hello' },
      { premium: true, sender: 'eugenethedream', text: 'hello' },
      { premium: false, sender: 'example-acct.testnet', text: 'Aloha' },
      [length]: 10
    ]

</p>
</details>

---

## Transactions

### `pex tx-status`

> Queries transaction status by hash and accountId.

-   arguments: `txHash` `--accountId`
-   options: `default`

**Example:**

```bash
pex tx-status FY8hBam2iyQfdHkdR1dp6w5XEPJzJSosX1wUeVPyUvVK --accountId guest-book.testnet
```

<details>
<summary><strong>Example Response</strong></summary>
<p>

```json
Transaction guest-book.testnet:FY8hBam2iyQfdHkdR1dp6w5XEPJzJSosX1wUeVPyUvVK
{
  status: { SuccessValue: '' },
  transaction: {
    signer_id: 'example-acct.testnet',
    public_key: 'ed25519:AXZZKnp6ZcWXyRNdy8FztYrniKf1qt6YZw6mCCReXrDB',
    nonce: 20,
    receiver_id: 'guest-book.testnet',
    actions: [
      {
        FunctionCall: {
          method_name: 'addMessage',
          args: 'eyJ0ZXh0IjoiQWxvaGEifQ==',
          gas: 300000000000000,
          deposit: '0'
        }
      },
      [length]: 1
    ],
    signature: 'ed25519:5S6nZXPU72nzgAsTQLmAFfdVSykdKHWhtPMb5U7duacfPdUjrj8ipJxuRiWkZ4yDodvDNt92wcHLJxGLsyNEsZNB',
    hash: 'FY8hBam2iyQfdHkdR1dp6w5XEPJzJSosX1wUeVPyUvVK'
  },
  transaction_outcome: {
    proof: [ [length]: 0 ],
    block_hash: '6nsjvzt6C52SSuJ8UvfaXTsdrUwcx8JtHfnUj8XjdKy1',
    id: 'FY8hBam2iyQfdHkdR1dp6w5XEPJzJSosX1wUeVPyUvVK',
    outcome: {
      logs: [ [length]: 0 ],
      receipt_ids: [ '7n6wjMgpoBTp22ScLHxeMLzcCvN8Vf5FUuC9PMmCX6yU', [length]: 1 ],
      gas_burnt: 2427979134284,
      tokens_burnt: '242797913428400000000',
      executor_id: 'example-acct.testnet',
      status: {
        SuccessReceiptId: '7n6wjMgpoBTp22ScLHxeMLzcCvN8Vf5FUuC9PMmCX6yU'
      }
    }
  },
  receipts_outcome: [
    {
      proof: [ [length]: 0 ],
      block_hash: 'At6QMrBuFQYgEPAh6fuRBmrTAe9hXTY1NzAB5VxTH1J2',
      id: '7n6wjMgpoBTp22ScLHxeMLzcCvN8Vf5FUuC9PMmCX6yU',
      outcome: {
        logs: [ [length]: 0 ],
        receipt_ids: [ 'FUttfoM2odAhKNQrJ8F4tiBpQJPYu66NzFbxRKii294e', [length]: 1 ],
        gas_burnt: 3559403233496,
        tokens_burnt: '355940323349600000000',
        executor_id: 'guest-book.testnet',
        status: { SuccessValue: '' }
      }
    },
    {
      proof: [ [length]: 0 ],
      block_hash: 'J7KjpMPzAqE7iX82FAQT3qERDs6UR1EAqBLPJXBzoLCk',
      id: 'FUttfoM2odAhKNQrJ8F4tiBpQJPYu66NzFbxRKii294e',
      outcome: {
        logs: [ [length]: 0 ],
        receipt_ids: [ [length]: 0 ],
        gas_burnt: 0,
        tokens_burnt: '0',
        executor_id: 'example-acct.testnet',
        status: { SuccessValue: '' }
      }
    },
    [length]: 2
  ]
}
```

</p>
</details>

---

## Global Options

| Option                      | Description                                                                                     |
|-----------------------------|-------------------------------------------------------------------------------------------------|
| `--help`                    | Show help  [boolean]                                                                            |
| `--version`                 | Show version number  [boolean]                                                                  |
| `-v, --verbose`             | Prints out verbose output  [boolean] [default: false]                                           |

> Got a question? <a href="https://stackoverflow.com/questions/tagged/pexprotocol"> <h8>Ask it on StackOverflow!</h8></a>

## License

This repository is distributed under the terms of both the MIT license and the Apache License (Version 2.0).
See [LICENSE](LICENSE) and [LICENSE-APACHE](LICENSE-APACHE) for details.

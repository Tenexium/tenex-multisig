# Tenexium Multisig CLI

Minimal Python CLI to interact with the on-chain `MultiSigWallet` contract.

- **Simple project structure**: `requirements.txt`, `multisig.py`, `.env` (user-provided), `.env.example`
- **Works with any RPC**: Bittensor public node, local node
- **No external build**: ABI is embedded; you can override with `--abi` when needed
- **Supports**: owner actions, view actions, and `submit`

## Prerequisites

- Python 3.8+

## Install

```bash

python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
```

## Configure environment

Copy and edit `.env`:

```bash
cp .env.example .env
```

Update values:

- `PRIVATE_KEY`: EOA to sign txs. Keep it secret.
- `RPC_URL`: HTTP endpoint (or use default)
- `CONTRACT_ADDRESS`: Deployed `MultiSigWallet` address (or Use default)
- `CHAIN_ID`: Network chain id (Or use default)

## Command overview

### Owner actions

- `python multisig.py confirm <transaction_id>`: Add your confirmation to a queued transaction
- `python multisig.py revoke <transaction_id>`: Revoke your confirmation (if not executed)
- `python multisig.py execute <transaction_id>`: Execute a confirmed transaction
- `python multisig.py add-owner <new_owner_address>`: Propose adding an owner (encodes and submits to the multisig)
- `python multisig.py remove-owner <old_owner_address>`: Propose removing an owner (encodes and submits to the multisig)
- `python multisig.py set-lock-period <lock_period>` (alias: `setlockperiod`): Propose setting the lock period in blocks (encodes and submits to the multisig)

### View actions

- `python multisig.py owners`: List all owners
- `python multisig.py threshold`: Current confirmation threshold
- `python multisig.py txcount`: Total number of submitted transactions
- `python multisig.py tx <transaction_id>`: Get a transaction by id
- `python multisig.py is-owner <owner_address>`: Check if an address is an owner
- `python multisig.py is-confirmed <transaction_id> <owner>`: Check if a tx is confirmed by an owner
- `python multisig.py confcount <transaction_id>`: Get confirmation count for a tx
- `python multisig.py owner-ver`: Current owner set version
- `python multisig.py lock-period`: Current lock period
- `python multisig.py submission-block <transaction_id>`: Block number when the tx was submitted

### Submit

- `python multisig.py submit --to <address> [--data 0x...] [--value-eth N | --value-wei N]`: Submit any transaction via the multisig (to another contract or to send ETH)

Global flags (apply to most commands):

- `--rpc`: Override `RPC_URL`
- `--contract`: Override `CONTRACT_ADDRESS`
- `--abi`: Path to ABI JSON (optional, defaults to embedded ABI)
- `--json`: JSON output

Signing/fee flags (for write/admin commands):

- `--pk`: Override `PRIVATE_KEY`
- `--from`: Sender override (defaults to `PRIVATE_KEY` address)
- `--chain-id`: Override `CHAIN_ID`
- `--gas`, `--nonce`
- `--gas-price-gwei` (legacy) or `--max-fee-gwei`/`--max-priority-gwei` (EIP-1559)
- `--no-wait`: Do not wait for receipt

- Encode calldata for a function:
  
```bash
DATA=$(python multisig.py --abi ./abi.json encode functionName functionArgs)
python multisig.py submit --to 0xTarget --data "$DATA"
```

- Submit calldata to another contract:
  
```bash
python multisig.py submit --to 0xTarget --data 0xdeadbeef
```

- Send ETH via the multisig:

```bash
python multisig.py submit --to 0xRecipient --value-eth 0.1
```

## Argument formatting

- **address**: `0x...`, checksum not required (auto-normalized)
- **uint256/int**: decimal or hex `0x...`
- **bool**: `true/false`, `1/0`, `yes/no`
- **bytes / bytesN**:
  - hex: `0x...`
  - ascii: `raw:hello`
- **arrays**: JSON array `["0xA", "0xB"]` or comma-separated `0xA,0xB`

## Fees and gas

- By default, the CLI uses EIP-1559 if available and falls back to legacy gas price
- You can force legacy with `--gas-price-gwei`, or specify `--max-fee-gwei` and `--max-priority-gwei`
- Gas is auto-estimated; you can override with `--gas`

## Troubleshooting

- "Missing RPC URL/contract address/private key" → Ensure `.env` is set or pass flags
- "Reverted" → Function call failed; check arguments, confirmations, and threshold
- "insufficient funds for gas" → Top up the signer address
- Chain ID mismatch → set `--chain-id` or `CHAIN_ID` to match the network

## Project structure

```js
.
├── multisig.py           # CLI
├── requirements.txt      # Dependencies
└── .env.example          # Sample env
```

## Security

- Never commit your real `PRIVATE_KEY`
- Prefer testing on a testnet before mainnet
- This CLI signs transactions locally; hardware wallets are not supported

## Subtensor EVM (TAO) quick config

Defaults for Subtensor EVM:

- Name: Subtensor EVM
- RPC URLs:
  - https://lite.chain.opentensor.ai
  - https://bittensor-lite-public.nodies.app/
- Chain ID: 964
- Token symbol: TAO
- Explorer: https://evm.taostats.io

Example `.env`:

```env
RPC_URL=https://lite.chain.opentensor.ai
PRIVATE_KEY=0x...
CONTRACT_ADDRESS=0xYourMultisigOnSubtensor
CHAIN_ID=964
```

Verify connection:

```bash
python multisig.py owners
```

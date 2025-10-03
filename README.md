# Tenex Multisig CLI

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

You can override any env via CLI flags (see below).

## Quick start

- Show owners:
  
```bash
python multisig.py owners
```

- Show threshold:
  
```bash
python multisig.py threshold
```

- Show transaction count and a tx:

```bash
python multisig.py txcount
python multisig.py tx 0
```

- Show lock period and submission block for a tx:
  
```bash
python multisig.py lock-period
python multisig.py submission-block 0
```

## Command overview

### Owner actions

- `confirm <id>`: Add your confirmation to a queued transaction
- `revoke <id>`: Revoke your confirmation (if not executed)
- `execute <id>`: Execute a confirmed transaction
- `add-owner <address>`: Propose adding an owner (encodes and submits to the multisig)
- `remove-owner <address>`: Propose removing an owner (encodes and submits to the multisig)
- `set-lock-period <period>` (alias: `setlockperiod`): Propose setting the lock period (encodes and submits to the multisig)

### View actions

- `owners`: List all owners
- `threshold`: Current confirmation threshold
- `txcount`: Total number of submitted transactions
- `tx <id>`: Get a transaction by id
- `is-owner <addr>`: Check if an address is an owner
- `is-confirmed <id> <owner>`: Check if a tx is confirmed by an owner
- `confcount <id>`: Get confirmation count for a tx
- `owner-ver`: Current owner set version
- `lock-period`: Current lock period
- `submission-block <id>`: Block number when the tx was submitted

### Submit

- `submit --to <address> [--data 0x...] [--value-eth N | --value-wei N]`: Submit any transaction via the multisig (to another contract or to send ETH)

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

## Examples

### Owner actions

- Confirm / revoke / execute a tx:
  
```bash
python multisig.py confirm 0
python multisig.py revoke 0
python multisig.py execute 0
```

- Add owner:
  
```bash
python multisig.py add-owner 0xNewOwnerAddress
```

- Remove owner:
  
```bash
python multisig.py remove-owner 0xOwnerToRemove
```

- Set lock period:
  
```bash
python multisig.py set-lock-period 14400
# or using the alias
python multisig.py setlockperiod 14400
```

### View actions

- Owners:
  
```bash
python multisig.py owners
```

- Threshold:
  
```bash
python multisig.py threshold
```

- Tx count and a specific tx:
  
```bash
python multisig.py txcount
python multisig.py tx 0
```

- Owner checks and confirmations:
  
```bash
python multisig.py is-owner 0xOwnerAddress
python multisig.py is-confirmed 0 0xOwnerAddress
python multisig.py confcount 0
python multisig.py owner-ver
```

- Lock period and submission block:
  
```bash
python multisig.py lock-period
python multisig.py submission-block 0
```

### Submit

- Encode calldata for a function (e.g., ERC20 transfer with external ABI):
  
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

Examples:

```bash
# bytes
python multisig.py submit --to 0xTarget --data 0x11223344
python multisig.py submit --to 0xTarget --data raw:ping
```

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

## License

MIT

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

# Pre-Merge Fork Support - Task Notes

## Goal
Add support for Pre-Merge forks (Frontier through GrayGlacier) to the ethrex Ethereum client.

## What Was Done (This Iteration)

### 1. ChainConfig - Added Pre-Merge Fork Detection
**File**: `crates/common/types/genesis.rs`

- Added activation check methods for all pre-merge forks:
  - `is_homestead_activated(block_number)`
  - `is_dao_fork_activated(block_number)`
  - `is_tangerine_activated(block_number)`
  - `is_spurious_dragon_activated(block_number)`
  - `is_byzantium_activated(block_number)`
  - `is_constantinople_activated(block_number)`
  - `is_petersburg_activated(block_number)`
  - `is_muir_glacier_activated(block_number)`
  - `is_berlin_activated(block_number)`
  - `is_arrow_glacier_activated(block_number)`
  - `is_gray_glacier_activated(block_number)`
  - `is_paris_activated(block_number)`

- Added new method `get_fork_for_block(block_number, block_timestamp)`:
  - Determines the correct fork using both block number (pre-merge) and timestamp (post-merge)
  - Returns the correct Fork enum value from Frontier through BPO5

### 2. Precompile Historical Activation
**File**: `crates/vm/levm/src/precompiles.rs`

Updated precompile activation forks to match Ethereum mainnet history:
- `ECRECOVER`, `SHA2_256`, `RIPEMD_160`, `IDENTITY`: Now activate at `Frontier` (was `Paris`)
- `MODEXP`, `ECADD`, `ECMUL`, `ECPAIRING`: Now activate at `Byzantium` (was `Paris`)
- `BLAKE2F`: Now activates at `Istanbul` (was `Paris`)

### 3. EF Tests Fork Configurations
**File**: `tooling/ef_tests/blockchain/fork.rs`

Added chain configurations for all pre-merge forks:
- `FRONTIER_CONFIG`
- `HOMESTEAD_CONFIG`
- `TANGERINE_CONFIG`
- `SPURIOUS_DRAGON_CONFIG`
- `BYZANTIUM_CONFIG`
- `CONSTANTINOPLE_CONFIG`
- `ISTANBUL_CONFIG`
- `BERLIN_CONFIG`
- `LONDON_CONFIG`

Updated `Fork::chain_config()` to return appropriate configs for pre-merge forks instead of panicking.

## What Still Needs to Be Done

### High Priority (Core Functionality)

1. **Update EVM to use `get_fork_for_block`**
   - `crates/vm/levm/src/environment.rs`: `EVMConfig::new_from_chain_config()` currently uses `chain_config.fork(block_header.timestamp)` which only supports post-merge forks. Should use `get_fork_for_block(block_number, timestamp)`.

2. **Update blockchain code to use new fork method**
   - `crates/blockchain/blockchain.rs:2105`: `current_fork()` method uses timestamp only
   - `crates/blockchain/payload.rs`: Fork detection in various places

3. **Fork-specific gas costs for pre-merge**
   - `crates/vm/levm/src/gas_cost.rs`: Some gas costs changed in pre-merge forks (e.g., EXP opcode cost changed in Spurious Dragon, SLOAD changed in Berlin)
   - Need to add fork checks for historical gas costs

4. **Fork-specific opcodes**
   - Some opcodes were introduced in pre-merge forks:
     - `DELEGATECALL`: Homestead
     - `STATICCALL`, `RETURNDATASIZE`, `RETURNDATACOPY`: Byzantium
     - `CREATE2`, `EXTCODEHASH`, `SHL`, `SHR`, `SAR`: Constantinople
     - Access list opcodes (warm/cold): Berlin
   - Need to validate opcode availability based on fork

### Medium Priority (Testing & Validation)

5. **Run EF tests for pre-merge forks**
   - The configs are added but tests need to be run to validate behavior
   - Command: `cargo test --manifest-path tooling/ef_tests/state/Cargo.toml`

6. **Add pre-merge fork support to state tests**
   - `tooling/ef_tests/state/` may need similar config updates

### Low Priority (Polish)

7. **Update `display_config()` in ChainConfig**
   - Currently only shows post-merge forks; could show pre-merge activation blocks too

8. **Add integration tests for pre-merge networks**
   - Test genesis file parsing for historical mainnet blocks

## Key Files Modified
- `crates/common/types/genesis.rs` - Core fork detection
- `crates/vm/levm/src/precompiles.rs` - Precompile activation
- `tooling/ef_tests/blockchain/fork.rs` - Test configurations

## Testing Commands
```bash
# Check compilation
cargo check --package ethrex-common --package ethrex-levm

# Run EF state tests (if available)
cargo test --manifest-path tooling/ef_tests/state/Cargo.toml
```

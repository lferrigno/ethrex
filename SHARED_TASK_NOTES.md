# Pre-Merge Fork Support - Task Notes

## Goal
Add support for Pre-Merge forks (Frontier through GrayGlacier) to the ethrex Ethereum client.

## Current Status
Core infrastructure for pre-merge fork support is now complete. All fork detection code has been updated to use `get_fork_for_block()` which properly handles both:
- Pre-merge forks (block number based): Frontier through GrayGlacier
- Post-merge forks (timestamp based): Paris through BPO5

## What Still Needs to Be Done

### High Priority (Core Functionality)

1. **Fork-specific gas costs for pre-merge**
   - `crates/vm/levm/src/gas_cost.rs`: Some gas costs changed in pre-merge forks:
     - EXP opcode cost changed in Spurious Dragon
     - SLOAD changed in Berlin (access list)
     - CALL/BALANCE etc. changed with Berlin access lists
   - Need to add fork checks for historical gas costs

2. **Fork-specific opcodes availability**
   - Some opcodes were introduced in pre-merge forks:
     - `DELEGATECALL`: Homestead
     - `STATICCALL`, `RETURNDATASIZE`, `RETURNDATACOPY`: Byzantium
     - `CREATE2`, `EXTCODEHASH`, `SHL`, `SHR`, `SAR`: Constantinople
     - Access list opcodes (warm/cold): Berlin
   - Need to validate opcode availability based on fork (return invalid opcode error for forks that don't support them)

### Medium Priority (Testing & Validation)

3. **Run EF tests for pre-merge forks**
   - Download vectors: `cd tooling/ef_tests/blockchain && make download-test-vectors`
   - Run tests: `make test-levm`
   - The fork configs are added but tests need to be run to validate behavior

4. **Add pre-merge fork support to state tests**
   - `tooling/ef_tests/state/` may need similar config updates if not already done

### Low Priority (Polish)

5. **Update `display_config()` in ChainConfig**
   - Currently only shows post-merge forks; could show pre-merge activation blocks too

## Key Files Modified
- `crates/common/types/genesis.rs` - Core fork detection (get_fork_for_block)
- `crates/vm/levm/src/environment.rs` - EVMConfig now uses get_fork_for_block
- `crates/vm/levm/src/precompiles.rs` - Precompile activation (Frontier, Byzantium, Istanbul)
- `crates/blockchain/blockchain.rs` - current_fork() uses get_fork_for_block
- `crates/blockchain/payload.rs` - create_payload uses get_fork_for_block
- `crates/vm/backends/mod.rs` - apply_system_calls uses get_fork_for_block
- `crates/vm/backends/levm/mod.rs` - prepare_block and extract_all_requests_levm
- `crates/networking/rpc/eth/transaction.rs` - estimate gas uses get_fork_for_block
- `crates/common/types/block.rs` - validate_excess_blob_gas uses get_fork_for_block
- `tooling/ef_tests/blockchain/fork.rs` - Test configurations for all forks

## Testing Commands
```bash
# Check compilation
cargo check --package ethrex-common --package ethrex-levm --package ethrex-blockchain --package ethrex-vm --package ethrex-rpc

# Download EF test vectors and run tests
cd tooling/ef_tests/blockchain
make download-test-vectors
make test-levm
```

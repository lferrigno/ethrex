# Pre-Merge Fork Support - Task Notes

## Goal
Add support for Pre-Merge forks (Frontier through GrayGlacier) to the ethrex Ethereum client.

## Current Status
**Fork-specific gas costs are now implemented!** The gas cost functions in `crates/vm/levm/src/gas_cost.rs` now properly handle pre-merge forks:

- **EXP opcode**: Uses 10 gas per byte pre-Spurious Dragon (EIP-160), 50 post-Spurious Dragon
- **SLOAD**: 50 gas (Frontier), 200 gas (Tangerine/EIP-150), warm/cold model (Berlin+)
- **BALANCE**: 20 gas (Frontier), 400 gas (Tangerine), 700 gas (Istanbul/EIP-1884), warm/cold model (Berlin+)
- **EXTCODESIZE**: 20 gas (Frontier), 700 gas (Tangerine), warm/cold model (Berlin+)
- **EXTCODECOPY**: 20 gas base (Frontier), 700 gas base (Tangerine), warm/cold model (Berlin+)
- **EXTCODEHASH**: 400 gas (Constantinople), 700 gas (Istanbul), warm/cold model (Berlin+)
- **CALL/CALLCODE/DELEGATECALL/STATICCALL**: 40 gas (Frontier), 700 gas (Tangerine), warm/cold model (Berlin+)
- **Subcall gas limit**: Full requested gas (pre-Tangerine), 63/64 rule (EIP-150, Tangerine+)

## What Still Needs to Be Done

### High Priority (Core Functionality)

1. **Fork-specific SSTORE gas costs**
   - The current SSTORE implementation uses EIP-2200/2929 gas costs
   - Pre-Istanbul forks had different SSTORE gas semantics:
     - Pre-Constantinople: Simple 5000 (modify) / 20000 (create) gas costs
     - Constantinople/Petersburg: EIP-1283 gas metering (partially reverted)
     - Istanbul+: EIP-2200 net gas metering
   - Also need to handle SSTORE refund differences across forks

### Medium Priority (Testing & Validation)

2. **Run EF tests for pre-merge forks**
   - Download vectors: `cd tooling/ef_tests/blockchain && make download-test-vectors`
   - Run tests: `make test-levm`
   - The fork configs are added but tests need to be run to validate behavior

3. **Add pre-merge fork support to state tests**
   - `tooling/ef_tests/state/` may need similar config updates if not already done

### Low Priority (Polish)

4. **Update `display_config()` in ChainConfig**
   - Currently only shows post-merge forks; could show pre-merge activation blocks too

## Key Files Modified
- `crates/common/types/genesis.rs` - Core fork detection (get_fork_for_block)
- `crates/vm/levm/src/environment.rs` - EVMConfig now uses get_fork_for_block
- `crates/vm/levm/src/precompiles.rs` - Precompile activation (Frontier, Byzantium, Istanbul)
- `crates/vm/levm/src/opcodes.rs` - Fork-specific opcode tables (Frontier through Osaka)
- `crates/vm/levm/src/gas_cost.rs` - **Fork-specific gas costs for EXP, SLOAD, BALANCE, EXTCODE*, CALL*s**
- `crates/vm/levm/src/opcode_handlers/arithmetic.rs` - op_exp passes fork to gas_cost::exp
- `crates/vm/levm/src/opcode_handlers/environment.rs` - op_balance, op_extcodesize, op_extcodecopy, op_extcodehash pass fork
- `crates/vm/levm/src/opcode_handlers/stack_memory_storage_flow.rs` - op_sload passes fork
- `crates/vm/levm/src/opcode_handlers/system.rs` - op_call, op_callcode, op_delegatecall, op_staticcall pass fork
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
cargo check --package ethrex-common --package ethrex-levm --package ethrex-blockchain --package ethrex-vm

# Download EF test vectors and run tests
cd tooling/ef_tests/blockchain
make download-test-vectors
make test-levm
```

## Gas Cost History Reference
| Fork | SLOAD | BALANCE | EXTCODESIZE | CALL base | EXP byte |
|------|-------|---------|-------------|-----------|----------|
| Frontier | 50 | 20 | 20 | 40 | 10 |
| Tangerine | 200 | 400 | 700 | 700 | 10 |
| Spurious Dragon | 200 | 400 | 700 | 700 | 50 |
| Istanbul | 800 | 700 | 700 | 700 | 50 |
| Berlin+ | warm/cold | warm/cold | warm/cold | warm/cold | 50 |

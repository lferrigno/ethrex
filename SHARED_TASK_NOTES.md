# Pre-Merge Fork Support - Task Notes

## Goal
Add support for Pre-Merge forks (Frontier through GrayGlacier) to the ethrex Ethereum client.

## Current Status
**COMPLETE - All tests pass!**

### Test Results (validated January 2026)
- **Blockchain tests**: 5,957 tests passed (0 failures)
  - Includes pre-merge forks: frontier (26), homestead (3), byzantium (1), constantinople (3), berlin (6), istanbul (6)
- **State tests**: 62,979 tests passed (0 failures)

### Implementation Summary
All core fork-specific gas costs and opcode behaviors are implemented:
- EXP opcode: 10 gas per byte pre-Spurious Dragon, 50 post-Spurious Dragon (EIP-160)
- SLOAD: 50 gas (Frontier), 200 gas (Tangerine), warm/cold model (Berlin+)
- BALANCE: 20 gas (Frontier), 400 gas (Tangerine), 700 gas (Istanbul), warm/cold model (Berlin+)
- EXTCODESIZE/EXTCODECOPY: 20 gas (Frontier), 700 gas (Tangerine), warm/cold model (Berlin+)
- EXTCODEHASH: 400 gas (Constantinople), 700 gas (Istanbul), warm/cold model (Berlin+)
- CALL/CALLCODE/DELEGATECALL/STATICCALL: 40 gas (Frontier), 700 gas (Tangerine), warm/cold model (Berlin+)
- Subcall gas limit: Full requested gas (pre-Tangerine), 63/64 rule (EIP-150, Tangerine+)
- SSTORE: Pre-Istanbul simple model, Istanbul+ EIP-2200 net gas metering, Berlin+ EIP-2929

## What Could Be Done (Polish, Low Priority)

1. **Update `display_config()` in ChainConfig**
   - Currently only shows post-merge forks; could show pre-merge activation blocks too

## Testing Commands
```bash
# Check compilation
cargo check --package ethrex-common --package ethrex-levm --package ethrex-blockchain --package ethrex-vm

# Run blockchain tests (from project root)
cd tooling/ef_tests/blockchain
make download-test-vectors  # first time only
make test-levm

# Run state tests (from project root)
cd tooling/ef_tests/state
make download-evm-ef-tests  # first time only
make test-levm
```

## Key Files
- `crates/common/types/genesis.rs` - Fork enum with all pre-merge forks, get_fork_for_block()
- `crates/vm/levm/src/gas_cost.rs` - Fork-specific gas costs
- `crates/vm/levm/src/opcodes.rs` - Fork-specific opcode availability
- `tooling/ef_tests/blockchain/fork.rs` - Test configurations for all forks

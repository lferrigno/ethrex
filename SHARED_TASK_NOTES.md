# Pre-Merge Fork Support - Task Notes

## Goal
Add support for Pre-Merge forks (Frontier through GrayGlacier) to the ethrex Ethereum client.

## Current Status
**Fork-specific opcode availability is now implemented.** The opcode table builder in `crates/vm/levm/src/opcodes.rs` now properly restricts opcodes based on fork:
- Frontier: Basic opcodes only (no DELEGATECALL, STATICCALL, CREATE2, etc.)
- Homestead: Adds DELEGATECALL
- Byzantium: Adds STATICCALL, RETURNDATASIZE, RETURNDATACOPY, REVERT
- Constantinople: Adds CREATE2, SHL, SHR, SAR, EXTCODEHASH
- Istanbul: Adds CHAINID, SELFBALANCE
- London: Adds BASEFEE
- Shanghai+: Adds PUSH0
- Cancun+: Adds MCOPY, TLOAD, TSTORE, BLOBHASH, BLOBBASEFEE
- Osaka+: Adds CLZ

## What Still Needs to Be Done

### High Priority (Core Functionality)

1. **Fork-specific gas costs for pre-merge**
   - `crates/vm/levm/src/gas_cost.rs`: Some gas costs changed in pre-merge forks:
     - EXP opcode cost changed in Spurious Dragon (EIP-160: 10 + 10 * byte_len -> 10 + 50 * byte_len)
     - SLOAD: Was 50 gas in Frontier, changed to 200 in Tangerine, then to access list model in Berlin
     - CALL/BALANCE/EXTCODE*: Changed with Berlin access lists (EIP-2929)
   - The current implementation uses post-Berlin gas costs (warm/cold access model)
   - Pre-Berlin forks should use simpler flat gas costs

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
- `crates/vm/levm/src/opcodes.rs` - **Fork-specific opcode tables (Frontier through Osaka)**
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

## Opcode Introduction by Fork Reference
| Fork | Opcodes Added |
|------|---------------|
| Frontier | All basic opcodes |
| Homestead | DELEGATECALL (0xF4) |
| Byzantium | STATICCALL (0xFA), RETURNDATASIZE (0x3D), RETURNDATACOPY (0x3E), REVERT (0xFD) |
| Constantinople | CREATE2 (0xF5), SHL (0x1B), SHR (0x1C), SAR (0x1D), EXTCODEHASH (0x3F) |
| Istanbul | CHAINID (0x46), SELFBALANCE (0x47) |
| London | BASEFEE (0x48) |
| Shanghai | PUSH0 (0x5F) |
| Cancun | MCOPY (0x5E), TLOAD (0x5C), TSTORE (0x5D), BLOBHASH (0x49), BLOBBASEFEE (0x4A) |
| Osaka | CLZ (0x1E) |

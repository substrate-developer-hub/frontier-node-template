# Using the [substrate-validator-set](https://github.com/gautamdhameja/substrate-validator-set) pallet

> Derived from the original [video instructions](https://www.youtube.com/watch?v=lIYxE-tOAdw)

## Start Nodes

In a new terminal for each command, in the working directory of the repo:

```bash
# Start Charlie:
./target/release/frontier-template-node --chain local --charlie --port 30336 --ws-port 9946 --base-path /tmp/local-testnet/c

# Start Bob:
./target/release/frontier-template-node --chain local --bob --port 30335 --ws-port 9945 --base-path /tmp/local-testnet/b

# Start Alice:
./target/release/frontier-template-node --chain local --alice --port 30335 --ws-port 9944 --base-path /tmp/local-testnet/a
```

> More options for the node are outlined in the
> [private network tutorial](https://substrate.dev/docs/en/tutorials/start-a-private-network/alicebob)

### Block Producers Logs

Validators that are _active and producing blocks_ will have log lines of the form:

```
2021-07-30 15:06:48 🙌 Starting consensus session on top of parent 0x7ecb8e86efd8f8839b223aa25826fe7bd0cb07ed157b1ad897583b8f248a0082
2021-07-30 15:06:48 🎁 Prepared block for proposing at 8 [hash: 0x84d81d71560a378dd97c54cfbebbf07fdd126f9f6fb22da344af5aea1d1819ed; parent_hash: 0x7ecb…0082; extrinsics (2): [0x54f7…cd72, 0x3765…dc03]]
2021-07-30 15:06:48 🔖 Pre-sealed block for proposal at 8. Hash now 0x6aca5bf9f5b29a85a36b0b6fec53a518228e096b9a4cc3b2e043f49e8befa50e, previously 0x84d81d71560a378dd97c54cfbebbf07fdd126f9f6fb22da344af5aea1d1819ed.
```

Notice `charlie` is _not_ producing blocks, as this `AccountID` was not set in the
[chain_spec.rs](../node/src/chain_spec.rs) genesis configuration as an authority

## Add `charlie` to Validator Set

Connect to `charlie` via the AppsUI. With the flags above we set port `9946`, so the node is
avalible at: https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9946#/

1. Add the custom types required in `Settings -> Developer`

```json
{
  "Keys": "SessionKeys2"
}
```

2. Rotate Charlie's Author Keys in `Developer -> RPC calls` and submit `author -> rotateKeys()`.
   Copy this new key, it will be hex. Something like:

```bash
# you *need* the new key from the rotateKeys() call, not this one
0xaa6ead94ba4e85efe9811fdfcaf3a49670ac2058d92c0fe44628219556aae716f2634efa97df1cb4a808c54a42344a8e2d702814ab231b6e082280af386803e1
```

3. Set Charlie's key in `Developer -> Extrinsics` and submit **Using Charlie's account**
   `session -> setKeys(keys,proof)`. Use the hex key you got in the last step, and set `proof` to
   `0x` or leave empty.

4. Use the `sudo` account (Alice) to add Chalie as a validator `Developer -> Extrinsics` and wrap
   with `sudo -> sudo(call)` with the call `validatorSet -> addValidator(validator_id)`, set CHARLIE
   here.

## Check Block Production

Check the `chalie` node is producing blocks now (once the next session starts):

- The node's logs include the lines [mentioned above](#block-producers-logs) now.
- Validate the chain state in the UI:
  - `Developer -> ChainState` and `session -> validators(): Vec<ValidatorId>` will have all
    three node's AccountIDs listed.
  - Also it is in `validatorSet -> validators(): Option<Vec<AccountId>>`.
- Look at the `Explorer` page and it lists the validator of blocks as they are produced, CHARLIE
  should appear.

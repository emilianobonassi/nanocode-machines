# nanocode-mpp on Boundless (testnet)

This fork adds a Boundless mode to `nanocode-mpp`: Codex requests go to a Boundless MPP endpoint and
are paid per request from your own wallet over a Tempo Moderato session (pathUSD). Upstream behavior
is unchanged when `BOUNDLESS_MPP_URL` is unset.

## Setup (4 commands)

```sh
cargo install --locked --git https://github.com/emilianobonassi/nanocode-machines --branch boundless --bin nanocode-mpp
mkdir -m 700 -p ~/.boundless-mpp && cast wallet new --json | jq -r '.[0].private_key' > ~/.boundless-mpp/wallet.key && chmod 600 ~/.boundless-mpp/wallet.key
ADDR=$(cast wallet address --private-key "$(cat ~/.boundless-mpp/wallet.key)") && cast rpc --rpc-url https://rpc.moderato.tempo.xyz tempo_fundAddress $ADDR && until [ "$(cast call --rpc-url https://rpc.moderato.tempo.xyz 0x20c0000000000000000000000000000000000000 'balanceOf(address)(uint256)' $ADDR | cut -d' ' -f1)" != 0 ]; do sleep 2; done
export BOUNDLESS_MPP_URL=https://api-s.inference.boundless.network/mpp   # staging; prod: https://api.inference.boundless.network/mpp
```

Then run `nanocode-mpp` (TUI) or `nanocode-mpp exec -m luna "…"`. The `/spend` panel shows what each
request cost, live, and the deposit left. Needs Rust 1.97+ and Foundry's `cast`.

## What Boundless mode changes

- Wallet: a plain secp256k1 key from `~/.boundless-mpp/wallet.key` (or `BOUNDLESS_MPP_KEY`), signing
  directly, instead of `tempo wallet login`. Channels persist in `~/.tempo/wallet/channels.db`.
- Network: Moderato (42431) and pathUSD by default (`BOUNDLESS_MPP_CHAIN_ID`, `BOUNDLESS_MPP_RPC_URL`).
- Model: nanocode's Codex models are rewritten to `BOUNDLESS_MPP_MODEL`
  (default `deepseek/deepseek-v4.1-flash-1017`); any `-m` Codex alias works.
- Only the Codex/Responses endpoint is configured; Mercator, the Tempo gateway and the tool proxy
  (mainnet services) are off.
- Deposit per channel: `NANOCODE_MPP_DEPOSIT` (default 1 pathUSD), topped up by the same amount.
- The spend panel shows model spend as channel deposits minus what is left, so it matches the
  server's charges exactly.

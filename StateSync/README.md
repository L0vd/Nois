# Nois State Sync

## Info
#### Public RPC endpoint: http://95.216.2.219:28657/
#### Public API: http://95.216.2.219:28317/

## Guide to sync your node using State Sync:

### Copy the entire command
```
sudo systemctl stop noisd
SNAP_RPC="http://95.216.2.219:28657"; \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.noisd/config/config.toml

peers="3aa7f66499a700625056b674bf26d4f457dc38da@95.216.2.219:28656" \
&& sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml 

noisd tendermint unsafe-reset-all --home ~/.noisd --keep-addr-book && sudo systemctl restart noisd && \
journalctl -u noisd -f --output cat
```

### Turn off State Sync Mode after synchronization
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.noisd/config/config.toml
```
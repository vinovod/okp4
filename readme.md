## RPC and API

RPC = http://86.48.3.186:26657/

API http://86.48.3.186:1317/

## StateSync

```python
RPC="86.48.3.186:26657"
RECENT_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((RECENT_HEIGHT - 500))
TRUST_HASH=$(curl -s "$RPC/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)
PEER="d8bf9b583a7492a0519bdfc3b17e44dd16991154@86.48.3.186:26656"
echo $RPC, $RECENT_HEIGHT, $TRUST_HEIGHT, $TRUST_HASH, $PEER
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.okp4d/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEER\"/" $HOME/.okp4d/config/config.toml
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = 1500/" $HOME/.okp4d/config/app.toml
systemctl stop okp4d
okp4d tendermint unsafe-reset-all --home /root/.okp4d --keep-addr-book
sudo systemctl restart okp4d && journalctl -u okp4d -f -o cat

```

## Snapshot

```python
cd $HOME
sudo systemctl stop okp4d
cp $HOME/.okp4d/data/priv_validator_state.json $HOME/.okp4d/priv_validator_state.json.backup
rm -rf $HOME/.okp4d/data/*
wget http://144.91.88.48:9000/okp4_snap.tar && tar -xvf okp4_snap.tar -C $HOME/.okp4d/data
rm -rf okp4_snap.tar
mv $HOME/.okp4d/priv_validator_state.json.backup $HOME/.okp4d/data/priv_validator_state.json
sudo systemctl restart okp4d && journalctl -u okp4d -f -o cat

```

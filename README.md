# nolus_snapshot


sudo systemctl stop nolusd

cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup 

nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book 
curl http://89.163.227.44/snap_nibiru.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nolus

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json 

sudo systemctl start nolusd
sudo journalctl -u nolusd -f --no-hostname -o cat










#state_sync


sudo systemctl stop nolusd

nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book 

peers="76a20bd1caebe528e3a6b09811a3b5befe6191fd@89.163.227.44:37656" 
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.defund/config/config.toml

SNAP_RPC=89.163.227.44:37656

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" ~/.nolusd/config/config.toml

sudo systemctl restart nolusd
sudo journalctl -u nolusd -f --no-hostname -o cat

# Instructions on adding a 2nd owner (hw-wallet) to functioning pool

WARNING: Once you are done, Transfer of funds from CLI wallet to HW wallet can ONLY be done after 2 snapshots, otherwise pool will not meet pledge in next epoch and no rewards would be paid

## Requirements

Install cardano-hw-cli in desktop
https://github.com/vacuumlabs/cardano-hw-cli


## STEP 1

Delegate hw-Wallet to pool in either Daedalus or Yoroi


## STEP 2 Export hw-Wallet public keys (stake) in desktop

```
cardano-hw-cli address key-gen
  --path 1852H/1815H/0H/2/0
  --verification-key-file hw-stake.vkey
  --hw-signing-file hw-stake.hwsfile
```


## STEP 3 Modify pool certificate to add hw-wallet (as owner and for rewards)

```
cardano-cli stake-pool registration-certificate \
  --cold-verification-key-file node-cold.vkey \
  --vrf-verification-key-file vrf.vkey \
  --pool-pledge <pledge> \
  --pool-cost <FixedFee> \
  --pool-margin <pool fee in fraction ie 0.011 for 1.1%> \
  --pool-reward-account-verification-key-file hw-stake.vkey \    <- Rewards go to hw-Wallet
  --pool-owner-stake-verification-key-file cli-stake.vkey \      <- previous CLI key
  --pool-owner-stake-verification-key-file hw-stake.vkey \       <- hw-wallet key
  --mainnet \
  --single-host-pool-relay <IP of public relay 1> --pool-relay-port <port 1> \
  --single-host-pool-relay <IP of public relay 2> --pool-relay-port <port 2> \
  --metadata-url <domain>/<path>/metadata.json \
  --metadata-hash <hash of metadata file> \
  --out-file pool.cert
```

Create a transaction tx-pool.raw that includes this pool certificate:
```
--certificate-file pool.cert
```

This transaction must be signed using witnesses (multi-sig)

we require 4 witnesses
  - node-cold.vkey
  - hw-stake.vkey
  - cli-stake.vkey
  - cli-payment (to pay for tx fees)


In offline server:

*node-cold*
```
cardano-cli transaction witness \
  --tx-body-file tx-pool.raw \
  --signing-key-file node-cold.skey \
  --mainnet \
  --out-file node-cold.witness
```

*cli-stake*
```
cardano-cli transaction witness \
  --tx-body-file tx-pool.raw \
  --signing-key-file cli-stake.skey \
  --mainnet \
  --out-file cli-stake.witness
  
```
*cli-payment*
```
cardano-cli transaction witness \
  --tx-body-file tx-pool.raw \
  --signing-key-file cli-payment.skey \
  --mainnet \
  --out-file cli-payment.witness
```

then we need to copy tx-pool.raw to desktop

Create a witness using my hw-stake.vkey in desktop (where ledger is connected)

*hw-stake*
```
cardano-hw-cli transaction witness
  --tx-body-file tx-pool.raw
  --hw-signing-file hw-stake.hwsfile
  --mainnet
  --out-file hw-stake.witness
```

Copy hw-stake.witness to offline server

Sign transaction with witnesses in offline server:
```
cardano-cli transaction assemble \
  --tx-body-file tx-pool.raw \
  --witness-file node-cold.witness \
  --witness-file cli-stake.witness \
  --witness-file cli-payment.witness \  
  --witness-file hw-stake.witness \
  --out-file tx-pool.multisign 
```

then submit tx-pool.multisign in a hot node
```
cardano-cli shelley transaction submit
```

... and Done !!! :+1:
<BR><BR>
***CRITICAL NOTE: Transfer of funds from CLI wallet to HW wallet can ONLY be done after 2 snapshots, otherwise pool will not meet pledge in next epoch and no rewards would be paid***

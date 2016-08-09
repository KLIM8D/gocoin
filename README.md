# GOcoin 

## Overview
A fork of [https://github.com/utamaro/gocoin](https://github.com/utamaro/gocoin)

This is a library to make bitcoin address and transactions which was initially forked from [hellobitcoin](https://github.com/prettymuchbryce/hellobitcoin),
and added some useful features.

GOcoin uses btcec library in [btcd](https://github.com/btcsuite/btcd) instead of https://github.com/toxeus/go-secp256k1
to make it a pure GO program.


## Features 

1. Normaly Payment(P2PKH) supporting multi TxIns and multi TxOuts.
2. Gethering unspent transaction outputs(UTXO) and send transactions by using [Blockr.io](https://blockr.io) WEB API.
3. M of N multisig whose codes were partially ported from https://github.com/soroushjp/go-bitcoin-multisig.

## Changes
1. Removed the "local" package btcec and changed the code to use [github.com/btcsuite/btcd/btcec](https://github.com/btcsuite/btcd/btcec)
2. Refactored a lot of code
3. Implements dynamic fee calculation using the API from [21.co](https://bitcoinfees.21.co/api)

## Requirements

This requires

* git
* go 1.5+


## Installation

    $ mkdir -p tmp/{src,bin,pkg}
    $ cd tmp
    $ export GOPATH=$(pwd)
    $ go get github.com/klim8d/gocoin


## Example
(This example omits error handlings for simplicity.)

## Key Handling

```go

import gocoin

func main(){
	//make a public and private key pair.
	key, _ := gocoin.GenerateKey(true)
	adr, _ := key.Pub.GetAddress()
	fmt.Println("address=", adr)
	wif := key.Priv.GetWIFAddress()
	fmt.Println("wif=", wif)
	
	//get key from wif
	wif := "928Qr9J5oAC6AYieWJ3fG3dZDjuC7BFVUqgu4GsvRVpoXiTaJJf"
	txKey, _ := gocoin.GetKeyFromWIF(wif)
}
```

## Normal Payment

```go
import gocoin

func main(){
	key, _ := gocoin.GenerateKey(true)

	//get unspent transactions
	service := gocoin.NewBlockrService(true)
	txs, _ := service.GetUTXO(adr,nil)
	
	//Normal Payment
	gocoin.Pay([]*Key{txKey}, []*gocoin.Amounts{&{gocoin.Amounts{"n2eMqTT929pb1RDNuqEnxdaLau1rxy3efi", 0.01*gocoin.BTC}}, service)
}
```

## M of N Multisig

```go
import gocoin

func main(){
	key, _ := gocoin.GenerateKey(true)
	service := gocoin.NewBlockrService(true)

	//2 of 3 multisig
	key1, _ := gocoin.GenerateKey(true)
	key2, _ := gocoin.GenerateKey(true)
	key3, _ := gocoin.GenerateKey(true)
	rs, _:= gocoin.NewRedeemScript(2, []*PublicKey{key1.Pub, key2.Pub, key3.Pub})
	//make a fund
	rs.Pay([]*Key{txKey}, 0.05*gocoin.BTC, service)

    //get a raw transaction for signing.
	rawtx, tx, _:= rs.CreateRawTransactionHashed([]*gocoin.Amounts{&{gocoin.Amounts{"n3Bp1hbgtmwDtjQTpa6BnPPCA8fTymsiZy", 0.05*gocoin.BTC}}, service)

	//spend the fund
	sign1, _:= key2.Priv.Sign(rawtx)
	sign2, _:= key3.Priv.Sign(rawtx)
	rs.Spend(tx, [][]byte{nil, sign1, sign2}, service)
}
```


# Contribution
Improvements to the codebase and pull requests are encouraged.


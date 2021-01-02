Contents
========
This document describes how to set up your own Signet network.

Launch Signet
=============

fujicoin.conf settings
----------------------
Set `fujicoin.conf` in the data folder(`~/.fujicoin`) referring to the example below.

    [main]
    #rpcuser=
    #rpcpassword=
    #rpcauth=user:ffb389c0.......0dd878972
    rpcallowip=127.0.0.1
    server=1
    
    [signet]
    #rpcuser=
    #rpcpassword=
    #rpcauth=user:ffb389c0.......0dd878972
    rpcallowip=127.0.0.1
    server=1
    daemon=1
    #addnode=
    #signetchallenge=512102f8........4f2a51ae

Start fujicoind with signet: `./fujicoind -signet`

Create a signetchallenge for your own network
---------------------------------------------
Follow the steps below to get the Fujicoin legacy address and pubkey.

    ./fujicoind -regtest -daemon
    ./fujicoin-cli -regtest createwallet "test"
    ADDR=$(./fujicoin-cli -regtest getnewaddress "" "legacy")
    echo $ADDR
    PRIVKEY=$(./fujicoin-cli -regtest dumpprivkey $ADDR)
    echo $PRIVKEY
    ./fujicoin-cli -regtest getaddressinfo $ADDR | grep pubkey
    # "pubkey": "THE_REAL_PUBKEY"
    ./fujicoin-cli -regtest stop

Your signetch allenge is below.

    signetchallenge = 5121 + THE_REAL_PUBKEY + 51ae
                    = 512102f8........4f2a51ae

Mining environment settings
---------------------------
Set the completed signetchallenge in fujicoin.conf and restart fujicoind. Then import the private key above.

    ./fujicoind -signet
    CLI="./fujicoin-cli -signet"
    $CLI createwallet "signet"
    $CLI -signet importprivkey $PRIVKEY

Find Signet derived magic in debug.log in Signet's data folder.

    cat ~/.fujicoin/signet/debug.log | grep magic

Get the Fujicoin mining utility from the Fujicoin download center, and extract it to the `fujicoin-cli` folder.
Change the value of `SIGNET_HEADER` on line 34 of `miner.py` to the value of Signet derived magic obtained above.

Mining
======

To mine the first block in your custom chain, you can run:

    cd to_fujicoind_folder
    CLI="./fujicoin-cli -signet"
    MINER="python3 miner.py"
    GRIND="./fujicoin-util grind"
    ADDR="W4FVbSKsCyRcMPm8pAsdSsgSAtiXAyfovj"  # Note: Use legacy address
    $MINER --cli="$CLI" generate --grind-cmd="$GRIND" --address="$ADDR" --set-block-time=-1 --nbits=1e00f403

This will mine a block with the current timestamp. If you want to backdate the chain, you can give a different timestamp to --set-block-time.

You will then need to pick a difficulty target. Since signet chains are primarily protected by a signature rather than proof of work, there is no need to spend as much energy as possible mining, however you may wish to choose to spend more time than the absolute minimum. The calibrate subcommand can be used to pick a target, eg:

    $MINER calibrate --grind-cmd="$GRIND"
    nbits=1e00f403 for 3m average mining time

It defaults to estimating an nbits value resulting in 3m average time to find a block, but the --seconds parameter can be used to pick a different target, or the --nbits parameter can be used to estimate how long it will take for a given difficulty.

Using the --ongoing parameter will then cause the signet miner to create blocks indefinitely. It will pick the time between blocks so that difficulty is adjusted to match the provided --nbits value.

    $MINER --cli="$CLI" generate --grind-cmd="$GRIND" --address="$ADDR" --nbits=1e00f403 --ongoing


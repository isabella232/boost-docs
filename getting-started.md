# As a storage provider

### Initialization and Running

1\. Make sure you have a Lotus node and miner running

2\. Create and send funds to two new wallets for Boost

Boost currently uses two wallets for storage deals:

* The publish storage deals wallet - This wallet pays the gas cost when Boost sends the `PublishStorageDeals` message.
* The deal collateral wallet - When the Storage Provider accepts a deal, they must put collateral for the deal into escrow. Boost moves funds from this wallet into escrow with the `StorageMarketActor`.

```
PUBLISH_STORAGE_DEALS_WALLET=`lotus wallet new bls`
PLEDGE_COLLAT_WALLET=`lotus wallet new bls`
lotus send --from mywallet $PUBLISH_STORAGE_DEALS_WALLET 10
lotus send --from mywallet $PLEDGE_COLLAT_WALLET 10
```

3\. Create and initialize the Boost repository

{% hint style="warning" %}
If you are already running a Lotus markets service process, you should\
run `boostd migrate` instead of `boostd init`

See section [migrate-from-a-split-markets-process-to-boost.md](migrate-from-a-split-markets-process-to-boost.md "mention") for more details.
{% endhint %}

Boost keeps all data in a directory called the repository. By default the repository is at `~/.boost`. To use a different location pass the `--boost-repo` parameter.

Create authentication tokens for the `boost` nodes. They will be used by the `boost` node to make JSON-RPC calls to the `mining/sealing/proving` node.

```
export APISEALER=`lotus-miner auth api-info --perm=admin`
export APISECTORINDEX=`lotus-miner auth api-info --perm=admin`
```

Run `boostd init` to create and initialize the repository:

```
boostd --vv init \
       --api-sealer=$APISEALER \
       --api-sector-index=$APISECTORINDEX \
       --wallet-publish-storage-deals=$PUBLISH_STORAGE_DEALS_WALLET \
       --wallet-collateral-pledge=$PLEDGE_COLLAT_WALLET \
       --max-staging-deals-bytes=50000000000
```

* `--api-sealer` is the API info for the lotus-miner instance that does sealing
* `--api-sector-index` is the API info for the lotus-miner instance that provides storage
* `--max-staging-deals-bytes` is the maximum amount of storage to be used for downloaded files (once the limit is reached Boost will reject subsequent incoming deals)

4\. Run the `boostd` service, which will start:

* libp2p listeners for storage and retrieval
* the JSON RPC API
* the graphql interface (used by the react front-end)
* the web server for the react front-end

```
boostd --vv run
```

{% hint style="info" %}
In your firewall you will need to open the ports that libp2p listens on, so that Boost can receive storage and retrieval deals.

See the `Libp2p` section of `config.toml` in the [architecture.md](architecture.md "mention")
{% endhint %}

### Web UI

1. Build the React frontend

```
cd react

# Download and install npm packages needed by the React frontend
npm install

# Build the optimized JavaScript and CSS in boost/react/build
npm run build
```

1. Open the Web UI

Open http://localhost:8080 in your browser.

{% hint style="info" %}
To access a web UI running on a remote server, you can open an SSH tunnel from your local machine:

```
ssh -L 8080:localhost:8080 myserver
```
{% endhint %}

##

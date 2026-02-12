# Archivist-docker-compose

Here's a copy of my Archivist setup. I use this to run a node in the devnet and help store Ethereum history data. To switch this from devnet to testnet (and later mainnet) should be rediculously simple to the point where I don't need to explain it. You DO have to keep an eye on the image: When new versions come up, you may want to update. It's possible to replace the sha-abcd with "latest", but I'm not sure I'd recommend that.

## archivist-node.yaml
```yaml
services:
  archivist-base:
    image: durabilitylabs/archivist-node:sha-b37bc60-dist-tests
    pull_policy: always
    environment:
      - NETWORK=devnet
      - ARCHIVIST_DATA_DIR=/data
      - ARCHIVIST_LOG_LEVEL=TRACE;warn:discv5,providers,manager,cache;warn:libp2p,multistream,switch,transport,tcptransport,semaphore,asyncstreamwrapper,lpstream,mplex,mplexchannel,noise,bufferstream,mplexcoder,secure,chronosstream,connection,connmanager,websock,ws-session,JSONRPC-HTTP-CLIENT,JSONRPC-CLIENT
      - ARCHIVIST_API_BINDADDR=0.0.0.0
      - ARCHIVIST_API_CORS_ORIGIN=*
      - ARCHIVIST_STORAGE_QUOTA=300gb
      - NAT_PUBLIC_IP_AUTO=https://ip.archivist.storage
      - ARCHIVIST_ETH_PROVIDER=https://rpc.devnet.archivist.storage
      - MARKETPLACE_ADDRESS_FROM_URL=https://config.archivist.storage/devnet/marketplace
      # Fast cleanup:
      # TTL 3 hours
      # 10000 blocks every minute
      - ARCHIVIST_BLOCK_TTL=10800
      - ARCHIVIST_BLOCK_MI=60
      - ARCHIVIST_BLOCK_MN=10000
      - ARCHIVIST_PERSISTENCE=true
      - ARCHIVIST_PROVER=true
      - ARCHIVIST_USE_SYSTEM_CLOCK=1
```

## docker-compose.yaml
```yaml
services:
  archivist-1:
    extends:
      file: archivist-node.yaml
      service: archivist-base
    environment:
      - ARCHIVIST_API_PORT=8080
      - ARCHIVIST_LISTEN_ADDRS=/ip4/0.0.0.0/tcp/8070
      - ARCHIVIST_DISC_PORT=8090
      - ETH_PRIVATE_KEY= ~~~ Your Eth Private key here, example: "0x123123123123..." ~~~ 
    volumes:
      - ./volumes/arch1-data:/data
    ports:
      - 8080:8080/tcp
      - 8070:8070/tcp
      - 8090:8090/udp

```

## Things you need to do
- Copy these files and same them on your server
- Check the ports specified in the docker-compose.yaml:
    - It is very important that 8070 and 8090 and exposed to the internet.
    - It is equally important that 8080 IS NOT exposed to the internet.
- Generate and save an Ethereum account. DO NOT USE your eth mainnet key, of course.
- Put the new private key in the "ETH_PRIVATE_KEY" field.
- For devnet/testnet, Get some free tokens:
    - Use the [Devnet Faucet](https://docs.archivist.storage/networks/devnet)
    - Use the [Testnet Faucet](https://docs.archivist.storage/networks/testnet#endpoints)

## Things you may want to do
- Choose different ports
- Adjust the storage quota
- Switch to a different network

## To start your node
- Run `docker compose up -d`
- Follow the logs: `docker compose logs -f`

## To start selling storage space and earning tokens
- You must create an "availability". This is an object the defines the terms for your storage space sales. (how much tokens you charge for it, how long the storage contracts can be, etc.)
- See [Archivist.Storage Docs](https://docs.archivist.storage/learn/using#create-storage-availability) for how to create an availability.

## Substreams on Solana Development Environment

We are going to run the Firehose stack here to locally index a development Solana validator producing blocks. This will give you a full blown locally Firehose instance on which you will be able to run your Substreams for development purposes.

> [!NOTE]
> Everything can be done through `docker compose` so you don't need to install all the tools, follow [Docker Compose](#docker-compose) instructions to follow that path.

You will need:
- [firecore](https://github.com/streamingfast/firehose-core/releases) binary available globally, with brew (Mac/Linux) do `brew install streamingfast/tap/firehose-core`.
- [firesol](https://github.com/streamingfast/firehose-solana/releases) binary available globally, with brew (Mac/Linux) do `brew install streamingfast/tap/firehose-solana`.
- [substreams](https://substreams.streamingfast.io/documentation/consume/installing-the-cli) CLI
- [solana-test-validator](https://docs.solanalabs.com/cli/install) to run a local Solana chain


### Instructions

1. Create a new folder that will hold the necessary files

   ```shell
   mkdir ./substreams-devenv
   cd ./substreams-devenv
   ```

1. Copy over the `firehose-core` sample config file `firehose-solana.yaml` in here:

   ```shell
   wget -O firehose-solana.yaml https://raw.githubusercontent.com/streamingfast/substreams-solana-devenv/master/firehose-solana.yaml
   ```

1. Run the `solana-test-validator` in one terminal:

   ```shell
   solana-test-validator
   ```

1. Run the `firehose-core` stack in another terminal:

   ```shell
   firecore -c ./firehose-solana.yaml start
   ```

1. Reach out to your local instance:

   ```shell
   substreams run -e localhost:9000 --plaintext https://github.com/streamingfast/substreams-sink-noop/raw/develop/substreams-head-tracker/substreams-head-tracker-v1.0.0.spkg -s 1 map_blocks
   ```

   > [!NOTE]
   > It takes ~40 blocks for Firehose to be be bootstrapped correctly, if you hit `Error: call sf.substreams.rpc.v2.Stream/Blocks: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing: dial tcp [::1]:9000: connect: connection refused`, wait a bit an retry

### Docker Compose

A `solana-test-validator` image is required to run inside Docker. Sadly, it's not easy to find one that work in all cases. The recommended approach is to locally compile a Docker image of `solana-test-validator` and use it. We provide a sample image here that works.

You first need to build a local version of `solana-test-validator` image with:

```shell
docker build -t solana-test-validator:v1.18.15 . -f Dockerfile.solana-test-validator
```

This step took ~10m on a Macbook M1 Pro, timings may differs.

> [!NOTE]
> Once the image is built, you don't need to repeat those steps anymore.

Once this is built and the image is available locally, you can simply launch:

```shell
docker compose up
```

This will launch both a `solana-test-validator` and a local Firehose instance that instruments this locally available Solana chain. You can then access the Docker Compose Substreams Tier1 with:

```shell
substreams run -e localhost:9000 --plaintext https://github.com/streamingfast/substreams-sink-noop/raw/develop/substreams-head-tracker/substreams-head-tracker-v1.0.0.spkg -s 1 map_blocks
```

> [!NOTE]
> It takes ~40 blocks for Firehose to be be bootstrapped correctly, if you hit `Error: call sf.substreams.rpc.v2.Stream/Blocks: rpc error: code = Unavailable desc = connection error: desc = "error reading server preface: read tcp [::1]:54885->[::1]:9000: read: connection reset by peer"`, wait a bit an retry


If you want to start from scratch again, do `docker compose down`
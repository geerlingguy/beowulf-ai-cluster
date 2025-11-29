# Beowulf AI Cluster

<p align="center"><img alt="Framework Mainboard beowulf AI cluster on desk" src="/resources/beowulf-cluster-framework-desktop.jpg" height="auto" width="600"></p>

With AI getting all the headlines and stock bubbles recently, I thought I'd create a project I can use to test out various distributed AI clustering tools on the various clusters I test.

## Installation and Usage

Make sure you have Ansible installed (`pip3 install ansible`), then copy the following files:

  - `cp example.hosts.ini hosts.ini`: This is an inventory of all the hosts in your cluster (or just a single computer).
  - `cp example.config.yml config.yml`: This has some configuration options you may need to override.

Each host should be reachable via SSH using the username set in `ansible_user`. Other Ansible options can be set under `[cluster:vars]` to connect in more exotic clustering scenarios (e.g. via bastion/jump-host).

Tweak other settings inside `config.yml` as desired.

> **Note**: The names of the nodes inside `hosts.ini` must match the hostname of their corresponding node.
> 
> For example, if you have `node-01.local` in your `hosts.ini` your host's hostname should be `node-01` and not something else like `raspberry-pi`.
>
> If you're testing with `.local` domains on Ubuntu, and local mDNS resolution isn't working, consider installing the `avahi-daemon` package:
>
> `sudo apt-get install avahi-daemon`

Then run the Ansible playbook inside this directory:

```
ansible-playbook main.yml
```

This will run two separate plays:

  1. Setup: downloads and compiles all the code required to run an AI model.
  2. Benchmark: Runs AI benchmarks, outputting the results in your console.

## Rebuilding llama.cpp

The quickest way to force a _rebuild_ of llama.cpp (like if you updated the `llama_build_opts` or want a later version) is to run:

```
ansible all -a "rm -rf /opt/llama.cpp" -b
```

Eventually I might make it easier to rebuild things automatically. But for now, this is simple, and it works.

## Benchmarks - Automated

There are multiple benchmarks included:

  - **llama.cpp benchmark - individual nodes**: Runs `llama-bench` on each node indepdently, and can be used to compare relative node performance (or to verify all your nodes have GPUs or NPUs recognized and utilized correctly). Performance results will be printed for each node individually.
  - **llama.cpp benchmark - full cluster (rpc-server)**: Configures llama.cpp in RPC mode, and runs a benchmark against the entire cluster. Performance results will be summarized for the entire cluster.
  - **distributed-llama benchmark - full cluster**: Configures distributed-llama workers on all but the root node, then runs a benchmark against the entire cluster.

To run benchmarks, run the playbook with the proper tag (run the individual benchmark at least one time first, to make sure everything is set up):

```
# For llama.cpp individual node benchmark:
ansible-playbook main.yml --tags llama-bench

# For llama.cpp full cluster benchmark:
ansible-playbook main.yml --tags llama-bench-cluster

# For distributed-llama full cluster benchmark:
ansible-playbook main.yml --tags dllama-bench-cluster
```

## Benchmarks - Manual

If you'd like to run a manual benchmark (e.g. to debug what's happening with something like `llama-bench` on a larger model), here's how to do it.

### llama.cpp RPC Manual Benchmark

  1. Launch `llama-rpc`: `ansible all -a "systemctl start llama-rpc" -b`
  2. Run a benchmark from one of the nodes (e.g. node 1):

```
cd /opt/llama.cpp
build/bin/llama-bench -v -m models/Llama-3.1-405B-Q4_K_M.gguf -n 128 -p 512 -pg 512,128 -ngl 125 -fa 1 -r 2 --rpc 10.0.2.233:50052,10.0.2.209:50052,10.0.2.242:50052,10.0.2.223:50052
```

You can grab all your node IP addresses with:

```
ansible all -m ansible.builtin.setup -a "filter=ansible_default_ipv4"
```

When you're finished, stop `llama-rpc`: `ansible all -a "systemctl stop llama-rpc" -b`

### distributed-llama Manual Benchmark

  1. Launch `dllama-worker` on all but the first node: `ansible all,\!framework-1.local -a "systemctl start dllama-worker" -b`
  2. Run a benchmark from the first node:

```
cd /opt/distributed-llama
./dllama inference \
  --model models/llama3_2_1b_instruct_q40/dllama_model_llama3_2_1b_instruct_q40.m \
  --tokenizer models/llama3_2_1b_instruct_q40/dllama_tokenizer_llama3_2_1b_instruct_q40.t \
  --buffer-float-type q80 \
  --prompt "Can you explain a hello world program" \
  --steps 256 \
  --max-seq-len 4096 \
  --nthreads 1 \
  --net-turbo 0 \
  --gpu-index 0 \
  --workers 10.0.2.209:9999 10.0.2.242:9999 10.0.2.223:9999
```

You can grab all your node IP addresses with:

```
ansible all -m ansible.builtin.setup -a "filter=ansible_default_ipv4"
```

When you're finished, stop `dllama-worker`: `ansible all -a "systemctl stop dllama-worker" -b`

## Exo Manual Benchmark

Because Exo's development seems to be at a standstill, the _only_ way I currently support Exo benchmarking is running it manually.

First, run the Exo setup playbook:

```
ansible-playbook main.yml --tags exo-setup
```

Then, log into each node and launch Exo (run `exo`)

Visit any node IP address or hostname in the browser using the port indicated in Exo's output, and you'll be greeted by a TinyChat UI. Start a chat to initiate a model download.

## Benchmark Results

_Currently_ I'm storing all benchmark results in my [ollama-benchmark](https://github.com/geerlingguy/ollama-benchmark?tab=readme-ov-file#findings) project.

I will eventually move cluster benchmarks into this repository, I think. I'm just lazy so they're all over there now.

## License

GPLv3

## Author

[Jeff Geerling](https://www.jeffgeerling.com).

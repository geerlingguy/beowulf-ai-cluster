# Beowulf AI Cluster

With AI getting all the headlines and stock bubbles recently, I thought I'd create a project I can use to test out various distributed AI clustering tools on the various clusters I test.

## Installation and Usage

## Benchmarking - Cluster

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

Then run the benchmarking playbook inside this directory:

```
ansible-playbook main.yml
```

This will run two separate plays:

  1. Setup: downloads and compiles all the code required to run an AI model.
  2. Benchmark: Runs AI benchmarks, outputting the results in your console.

## License

AGPLv3 or later.

## Author

[Jeff Geerling](https://www.jeffgeerling.com).

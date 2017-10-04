DNS Queue - A Parallelised DNS Prober
=================================

## What is DNS Parallel Prober?
This is a parallelised domain name prober to find as many subdomains of
a given domain as fast as possible.

**PLEASE NOTE** this script probes DNS servers actively, so please use
at your own risk. You are likely to get blacklisted and/or saturate your
bandwidth. Whatever you do, it's your responsibility to make sure you
have approval for it.

Hat tip to: [Kyle F.](https://github.com/radman404) for the original
idea and to [ZephrFish](https://github.com/ZephrFish) for all
improvements and testing.

![Demo Screenshot](screenshot.png?raw=true "Usage example")

### Quickstart

Install the requirements:

    pip install -r requirements.txt
    # if you don't want to install stuff as root, do:
    # virtualenv venv
    # source venv/bin/activate
    # pip install -r requirements.txt

Scan all 3-characters subdomains of `example.com` using 100 threads,
saving the results in `out.txt`:

    ./dns-queue.py example.com 100 out.txt

If you want to read the subdomains from a list, do:

    ./dns-queue.py example.com 100 out.txt -i subdomains.txt

By default the script uses the authoritative NS servers for the given
domain. To use different DNS servers:

    ./dns-queue.py example.com 100 out.txt -i subdomains.txt -n ns1.example.com, -n ns2.example.com

For help and other options (e.g. subdomain length, DNS timeouts, etc.):

    ./dns-queue.py -h

To stop: press `ctrl-c` - it will wait for the last threads to finish
and *only then* write all results to `out.txt`. Pressing `ctrl-c` twice
will likely not save anything to disk.

### Why 100 threads?

The optimal number of threads depends on:

 * How fast is the uplink
 * How quickly the domain name servers respond to queries

The value is best set empirically: on the same server, try with a value
of 100 and keep doubling it until it doesn't go any faster.

In more details, the script creates N threads (here N is 100) and gives
each thread a domain name to resolve. It then sleeps a small amount of
time before checking whether each thread is done or not.  The 'sleep
time' is adjusted depending on how fast the threads resolve.

Since network I/O is orders of magnitude slower than CPU I/O, **the
number of threads should not be limited by the number of cores**. In
other words: each thread will spend most of its time waiting for a DNS
response; that "idle time" can safely be used by other threads.

### Installation

    git clone .....
    cd dns-parallel-prober
    # if debian/ubuntu:
    sudo apt-get install python-virtualenv python-pip
    # create virtualenv to install the required python libs
    virtualenv venv
    # activate it
    source venv/bin/activate
    pip install dnspython
    # to deactivate the virtualenv run:
    # deactivate

### Features

- Ability to use raw brute-force of to load a list of subdomains
- Single Core parallel probing

### Multi-core scenario

If you have *lots* of cores and you can send out data faster than your CPU can fork threads and you want to max out your machine then the simplest solution is:

 1. Split a subdomain list into N files (with `N` matching the number of cores you want to use)
 2. Run this executable N times perhaps using `tmux`, feeding it each
    file
 3. Make sure you use a different output for each process
 4. ...
 5. Profit!

Alternatively, fork this repo and write a multiprocessing version. Good
luck.


### Demo Video
[![asciicast](https://asciinema.org/a/16teprhj9hykzrl8hmtyrte2k.png)](https://asciinema.org/a/16teprhj9hykzrl8hmtyrte2k)

## Notes

The key thing is that the iteration frequency is dynamically adapted to the depletion speed, i.e. the faster the threads complete the sooner new ones will be added until an equilibrium is reached. The tool tries to stay reasonably close to the maximum value set.

File `subdomains.txt` gathered from [research](http://haxpo.nl/haxpo2015ams/wp-content/uploads/sites/4/2015/04/D1-P.-Mason-K.-Flemming-A.-Gill-All-Your-Hostnames-Are-Belong-to-Us.pdf) carried out in 2014/15.

### Why threads and not processes

Because in this scenario the bottleneck is the network, not the CPU. I'm happy to be proven wrong! Just fork this repo and submit a pull request and some empirical data to back your claim.

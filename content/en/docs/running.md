---
title: Running 
description: Operating and Managing Falco
weight: 4
---


## Run Falco as a service

If you installed Falco by using [the deb or the rpm](../installation) package, you can start the service:

```bash
service falco start
```

You can also view the Falco logs using `journalctl`.

```bash 
journalctl -fu falco
```

## Run Falco manually

If you'd like to run Falco by hand, here's the full usage description for Falco:

```
Usage: falco [options]

Options:
 -h, --help                    Print this page
 -c                            Configuration file (default /source/falco/falco.yaml, /etc/falco/falco.yaml)
 -A                            Monitor all events, including those with EF_DROP_SIMPLE_CONS flag.
 -b, --print-base64            Print data buffers in base64.
                               This is useful for encoding binary data that needs to be used over media designed to.
 --cri <path>                  Path to CRI socket for container metadata.
                               Use the specified socket to fetch data from a CRI-compatible runtime.
 -d, --daemon                  Run as a daemon.
 --disable-cri-async           Disable asynchronous CRI metadata fetching.
                               This is useful to let the input event wait for the container metadata fetch
                               to finish before moving forward. Async fetching, in some environments leads
                               to empty fields for container metadata when the fetch is not fast enough to be
                               completed asynchronously. This can have a performance penalty on your environment
                               depending on the number of containers and the frequency at which they are created/started/stopped
 --disable-source <event_source>
                               Disable a specific event source.
                               Available event sources are: syscall, k8s_audit.
                               It can be passed multiple times.
                               Can not disable both the event sources.
 -D <substring>                Disable any rules with names having the substring <substring>. Can be specified multiple times.
                               Can not be specified with -t.
 -e <events_file>              Read the events from <events_file> (in .scap format for sinsp events, or jsonl for
                               k8s audit events) instead of tapping into live.
 -k <url>, --k8s-api <url>
                               Enable Kubernetes support by connecting to the API server specified as argument.
                               E.g. "http://admin:password@127.0.0.1:8080".
                               The API server can also be specified via the environment variable FALCO_K8S_API.
 -K <bt_file> | <cert_file>:<key_file[#password]>[:<ca_cert_file>], --k8s-api-cert <bt_file> | <cert_file>:<key_file[#password]>[:<ca_cert_file>]
                               Use the provided files names to authenticate user and (optionally) verify the K8S API server identity.
                               Each entry must specify full (absolute, or relative to the current directory) path to the respective file.
                               Private key password is optional (needed only if key is password protected).
                               CA certificate is optional. For all files, only PEM file format is supported. 
                               Specifying CA certificate only is obsoleted - when single entry is provided 
                               for this option, it will be interpreted as the name of a file containing bearer token.
                               Note that the format of this command-line option prohibits use of files whose names contain
                               ':' or '#' characters in the file name.
 -L                            Show the name and description of all rules and exit.
 -l <rule>                     Show the name and description of the rule with name <rule> and exit.
 --list [<source>]             List all defined fields. If <source> is provided, only list those fields for
                               the source <source>. Current values for <source> are "syscall", "k8s_audit"
 -m <url[,marathon_url]>, --mesos-api <url[,marathon_url]>
                               Enable Mesos support by connecting to the API server
                               specified as argument. E.g. "http://admin:password@127.0.0.1:5050".
                               Marathon url is optional and defaults to Mesos address, port 8080.
                               The API servers can also be specified via the environment variable FALCO_MESOS_API.
 -M <num_seconds>              Stop collecting after <num_seconds> reached.
 -N                            When used with --list, only print field names.
 -o, --option <key>=<val>      Set the value of option <key> to <val>. Overrides values in configuration file.
                               <key> can be a two-part <key>.<subkey>
 -p <output_format>, --print <output_format>
                               Add additional information to each falco notification's output.
                               With -pc or -pcontainer will use a container-friendly format.
                               With -pk or -pkubernetes will use a kubernetes-friendly format.
                               With -pm or -pmesos will use a mesos-friendly format.
                               Additionally, specifying -pc/-pk/-pm will change the interpretation
                               of %container.info in rule output fields.
 -P, --pidfile <pid_file>      When run as a daemon, write pid to specified file
 -r <rules_file>               Rules file/directory (defaults to value set in configuration file, or /etc/falco_rules.yaml).
                               Can be specified multiple times to read from multiple files/directories.
 -s <stats_file>               If specified, write statistics related to falco's reading/processing of events
                               to this file. (Only useful in live mode).
 --stats_interval <msec>       When using -s <stats_file>, write statistics every <msec> ms.
                               This uses signals, so don't recommend intervals below 200 ms.
                               Defaults to 5000 (5 seconds).
 -S <len>, --snaplen <len>
                               Capture the first <len> bytes of each I/O buffer.
                               By default, the first 80 bytes are captured. Use this
                               option with caution, it can generate huge trace files.
 --support                     Print support information including version, rules files used, etc. and exit.
 -T <tag>                      Disable any rules with a tag=<tag>. Can be specified multiple times.
                               Can not be specified with -t.
 -t <tag>                      Only run those rules with a tag=<tag>. Can be specified multiple times.
                               Can not be specified with -T/-D.
 -U,--unbuffered               Turn off output buffering to configured outputs.
                               This causes every single line emitted by falco to be flushed,
                               which generates higher CPU usage but is useful when piping those outputs
                               into another process or into a script.
 -V, --validate <rules_file>   Read the contents of the specified rules(s) file and exit.
                               Can be specified multiple times to validate multiple files.
 -v                            Verbose output.
 --version                     Print version number.
```


## Run within Docker {#docker}

This is how the Falco userspace process can be ran in a container. 

Once the kernel module has been installed directly on the host system, it can be used from within a container.

1. Install the kernel module:

    - You can use an official [installation method](/docs/installation) directly on the host
    - Alternatively, you can temporarily use a privileged container to install the driver on the host:

    ```shell
    docker pull falcosecurity/falco-driver-loader:latest
    docker run --rm -i -t \
        --privileged \
        -v /root/.falco:/root/.falco \
        -v /proc:/host/proc:ro \
        -v /boot:/host/boot:ro \
        -v /lib/modules:/host/lib/modules:ro \
        -v /usr:/host/usr:ro \
        -v /etc:/host/etc:ro \
        falcosecurity/falco-driver-loader:latest
    ``` 

2. Run Falco in a container using Docker with the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege):

    ```shell
    docker pull falcosecurity/falco-no-driver:latest
    docker run --rm -i -t \
        --cap-add SYS_PTRACE --pid=host $(ls /dev/falco* | xargs -I {} echo --device {}) \
        -v /var/run/docker.sock:/var/run/docker.sock \
        falcosecurity/falco-no-driver:latest
    ```


## Hot Reload

This will reload the Falco configuration and restart the engine without killing the pid. This is useful to propagate new config changes without killing the daemon.

```bash
kill -1 $(cat /var/run/falco.pid)
```
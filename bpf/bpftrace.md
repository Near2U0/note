
reference:https://github.com/iovisor/bpftrace/blob/master/INSTALL.md



On Ubuntu 16.04 and later, bpftrace is also available as a snap package (https://snapcraft.io/bpftrace), however,** the snap provides extremely limited file permissions so the --devmode option should be specified on installation in order avoid file access issues.**

```shell

sudo snap install --devmode bpftrace
sudo snap connect bpftrace:system-trace

```

The snap package also currently has issues with uprobes (#829).

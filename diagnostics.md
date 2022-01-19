### uptime

This is a quick way to view the load averages, which indicate the number of tasks (processes) wanting to run. On Linux systems, these numbers include processes wanting to run on CPU, as well as processes blocked in uninterruptible I/O (usually disk I/O). This gives a high level idea of resource load (or demand), but can’t be properly understood without other tools. Worth a quick look only.

    $ uptime
    23:51:26 up 21:31, 1 user, load average: 30.02, 26.43, 19.02

### dmesg | tail

This views the last 10 system messages, if there are any. Look for errors that can cause performance issues. The example above includes the oom-killer, and TCP dropping a request. Don’t miss this step! dmesg is always worth checking.

    $ dmesg | tail
    [1880957.563150] perl invoked oom-killer: gfp_mask=0x280da, order=0, oom_score_adj=0

### vmstat 1

Short for virtual memory stat

    $ vmstat 1
    procs ---------memory---------- ---swap-- -----io---- -system-- ------cpu-----
    r  b swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
    34  0    0 200889792  73708 591828    0    0     0     5    6   10 96  1  3  0  0

* r: Number of processes running on CPU and waiting for a turn. This provides a better signal than load averages for determining CPU saturation, as it does not include I/O. To interpret: an “r” value greater than the CPU count is saturation.
* free: Free memory in kilobytes. If there are too many digits to count, you have enough free memory. The “free -m” command, included as command 7, better explains the state of free memory.
* si, so: Swap-ins and swap-outs. If these are non-zero, you’re out of memory.
* us, sy, id, wa, st: These are breakdowns of CPU time, on average across all CPUs. They are user time, system time (kernel), idle, wait I/O, and stolen time (by other guests, or with Xen, the guest’s own isolated driver domain).

### mpstat -P ALL 1

This command prints CPU time breakdowns per CPU, which can be used to check for an imbalance. A single hot CPU can be evidence of a single-threaded application.

    $ mpstat -P ALL 1
    Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015  _x86_64_ (32 CPU)

    07:38:49 PM  CPU   %usr  %nice   %sys %iowait   %irq  %soft  %steal  %guest  %gnice  %idle
    07:38:50 PM  all  98.47   0.00   0.75    0.00   0.00   0.00    0.00    0.00    0.00   0.78

### pidstat 1

Pidstat is a little like top’s per-process summary, but prints a rolling summary instead of clearing the screen. This can be useful for watching patterns over time, and also recording what you saw (copy-n-paste) into a record of your investigation.

    $ pidstat 1
    Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015    _x86_64_    (32 CPU)

    07:41:02 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
    07:41:03 PM     0         9    0.00    0.94    0.00    0.94     1  rcuos/0

### iostat -xz 1

This is a great tool for understanding block devices (disks), both the workload applied and the resulting performance. Look for:
* r/s, w/s, rkB/s, wkB/s: These are the delivered reads, writes, read Kbytes, and write Kbytes per second to the device. Use these for workload characterization. A performance problem may simply be due to an excessive load applied.
* await: The average time for the I/O in milliseconds. This is the time that the application suffers, as it includes both time queued and time being serviced. Larger than expected average times can be an indicator of device saturation, or device problems.
* avgqu-sz: The average number of requests issued to the device. Values greater than 1 can be evidence of saturation (although devices can typically operate on requests in parallel, especially virtual devices which front multiple back-end disks.)
* %util: Device utilization. This is really a busy percent, showing the time each second that the device was doing work. Values greater than 60% typically lead to poor performance (which should be seen in await), although it depends on the device. Values close to 100% usually indicate saturation.


    $ iostat -xz 1
    Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015  _x86_64_ (32 CPU)

    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
    73.96    0.00    3.73    0.03    0.06   22.21

    Device:   rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    xvda        0.00     0.23    0.21    0.18     4.52     2.08    34.37     0.00    9.98   13.80    5.42   2.44   0.09

### sar -n DEV 1

Use this tool to check network interface throughput: rxkB/s and txkB/s, as a measure of workload, and also to check if any limit has been reached. In the above example, eth0 receive is reaching 22 Mbytes/s, which is 176 Mbits/sec (well under, say, a 1 Gbit/sec limit).

    $ sar -n DEV 1
    Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015     _x86_64_    (32 CPU)

    12:16:48 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
    12:16:49 AM      eth0  18763.00   5032.00  20686.42    478.30      0.00      0.00      0.00      0.00

### sar -n TCP,ETCP 1

    $ sar -n TCP,ETCP 1
    Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015    _x86_64_    (32 CPU)

    12:17:19 AM  active/s passive/s    iseg/s    oseg/s
    12:17:20 AM      1.00      0.00  10233.00  18846.00

This is a summarized view of some key TCP metrics. These include:
* active/s: Number of locally-initiated TCP connections per second (e.g., via connect()).
* passive/s: Number of remotely-initiated TCP connections per second (e.g., via accept()).
* retrans/s: Number of TCP retransmits per second.
# shell-samples
Some useful samples for making cli tools using shell scripts.

For authoring stand alone scripts.
Rather than solve the general problem, choosing instead to emphasize 
"simple" and "self contained" snippets.

# Setup
```bash
brew bundle
```
```
Installing flock
Installing shellcheck
Homebrew Bundle complete! <N> Brewfile dependencies now installed.
```

# Linting
```bash
shellcheck bin/*
```

# Use Cases
Prototyping, bootstrapping and other esoteric / constrained environments.

CLIs are best done with proper flag parsing libraries.
This repos strives to demonstrate solutions that are a reasonable compromise.

## Mac OS / bash 3.2x (2007) compatible script samples
- boiler plate, shebang line and shell options
- resolve script to absolute, non symlink file location
- handle short/long options (with or without '=') and positional params
- concurrent work queue
- progess bar w/percentage
- spinner
- resumable state files
- log stdout, stderr to a file while preserving terminal output

### Specifics
<details>
  <summary>boiler plate, shebang line and shell options</summary>

This is the basic, "sane" new file starting point.

Usage:
```
bin/strict-mode
```
```
bin/strict-mode -> error (set -e), line=6, command=false
```

Inspired by:
* http://redsymbol.net/articles/unofficial-bash-strict-mode/ (Aaron Maxwell)
  * https://olivergondza.github.io/2019/10/01/bash-strict-mode.html (Oliver Gondža)
  * https://disconnected.systems/blog/another-bash-strict-mode/ (Michael Daffin)
  * https://gist.github.com/robin-a-meade/58d60124b88b60816e8349d1e3938615 (Robin A. Meade)
  * https://unix.stackexchange.com/questions/674037/when-does-inherit-errexit-not-work
</details>

<details>
  <summary>resolve script to absolute, non symlink file location</summary>

Useful for logging the absolute script path and 
locating other related files with relative references.

Usage:
```
bin/resolved-path
```
```
/Users/<user>/src/github.com/cicdenv/shell-samples/bin/resolved-paths 
```

Inspired by:
* https://stackoverflow.com/questions/3915040/how-to-obtain-the-absolute-path-of-a-file-via-shell-bash-zsh-sh
  * `man pwd`
    ```
    -L  Display the logical current working directory.

    -P  Display the physical current working directory (all symbolic links resolved).
    ```
</details>

<details>
  <summary>handle short/long options (with or without '=') and positional params</summary>

Adds the right level of polish to scripts, help+usage and proper long options.

NOTEs:
* not using `getopt`, grouped short options aren't support - I'm fine with this since it simplifies the args processing
* using a manually crafted usage message and verifications - deliberately

Usage:
```
bin/options

bin/options -c

bin/options "dev:us-east-1" -c        "services" "dev:us-west-2"
bin/options "dev:us-east-1" --cluster "services" "dev:us-west-2"
bin/options "dev:us-east-1" --cluster="services" "dev:us-west-2"

bin/options "dev:us-east-1" --cluster="services" "dev:us-west-2" --dry-run

bin/options --cluster="services"

bin/options "dev:us-east-1" "dev:us-west-2"

bin/options -h
bin/options --help
```
```
Missing required arguments.

Usage:

  bin/options -c|--cluster=<cluster> [-d|--dry-run] [-h|--help] <account:region>+
```
```
-c requires an argument

Usage:

  options -c|--cluster=<cluster> [-d|--dry-run] [-h|--help] <account:region>+
```
```
bin/options

  cluster -> services
  args    -> dev:us-east-1 dev:us-west-2
  dry-run -> false
```
```
bin/options

  cluster -> services
  args    -> dev:us-east-1 dev:us-west-2
  dry-run -> true
```
```
bin/options

  cluster -> services
  args    -> 
  dry-run -> false

Missing required target(s): <account:region>+.

Usage:

  bin/options -c|--cluster=<cluster> [-d|--dry-run] [-h|--help] <account:region>+
```
```
bin/options

  cluster -> 
  args    -> dev:us-east-1 dev:us-west-2
  dry-run -> false

Missing required argument: cluster.

Usage:

  bin/options -c|--cluster=<cluster> [-d|--dry-run] [-h|--help] <account:region>+
```
```
Usage:

  options [-c|--cluser <cluster>] [-d|--dry-run] [-h|--help]
```

Inspired by:
* https://stackoverflow.com/questions/402377/using-getopts-to-process-long-and-short-command-line-options
</details>

<details>
  <summary>concurrent work queue</summary>

Multiple concurrent workers pulling work items from a shared queue.

NOTE: needs `flock` command line tool

Usage:
```
bin/work-queue
```
```
Coordination Files:
  start        -> /var/folders/9_/b5b9n2zn1s704g643lvj0db80000gn/T/start-XXXX.wORbaxtp
  start (lock) -> /var/folders/9_/b5b9n2zn1s704g643lvj0db80000gn/T/lock-XXXX.2Y3stJfJ
  fifo         -> /var/folders/9_/b5b9n2zn1s704g643lvj0db80000gn/T/fifo-XXXX.qNkD2iBa
  fifo (lock)  -> /var/folders/9_/b5b9n2zn1s704g643lvj0db80000gn/T/lock-XXXX.SWvvF8JZ

 1: queue worker started.
 2: queue worker started.
 7: queue worker started.
 6: queue worker started.
 5: queue worker started.
11: queue worker started.
 8: queue worker started.
 4: queue worker started.
10: queue worker started.
 9: queue worker started.
12: queue worker started.
 3: queue worker started.

All queue workers ready.

=> Enqueueing item: id=1, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  1: processing received queued item, id=1, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=2, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  2: processing received queued item, id=2, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=3, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  7: processing received queued item, id=3, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=4, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=5, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  6: processing received queued item, id=4, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  5: processing received queued item, id=5, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=6, item=2024-09-18_13:08:45:N_PDT_1726690125
<= 11: processing received queued item, id=6, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=7, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  8: processing received queued item, id=7, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=8, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  4: processing received queued item, id=8, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=9, item=2024-09-18_13:08:45:N_PDT_1726690125
<= 10: processing received queued item, id=9, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=10, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  9: processing received queued item, id=10, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=11, item=2024-09-18_13:08:45:N_PDT_1726690125
<= 12: processing received queued item, id=11, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=12, item=2024-09-18_13:08:45:N_PDT_1726690125
<=  3: processing received queued item, id=12, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
=> Enqueueing item: id=13, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=14, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=15, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=16, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=17, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=18, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=19, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=20, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=21, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=22, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=23, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=24, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=25, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=26, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=27, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=28, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=29, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=30, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=31, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=32, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=33, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=34, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=35, item=2024-09-18_13:08:45:N_PDT_1726690125
=> Enqueueing item: id=36, item=2024-09-18_13:08:45:N_PDT_1726690125
** completed: worker-id=1, item=1
** completed: worker-id=7, item=3
<=  1: processing received queued item, id=13, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  7: processing received queued item, id=14, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=6, item=4
** completed: worker-id=11, item=6
<=  6: processing received queued item, id=15, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=4, item=8
** completed: worker-id=10, item=9
<= 11: processing received queued item, id=16, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  4: processing received queued item, id=17, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<= 10: processing received queued item, id=18, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=12, item=11
<= 12: processing received queued item, id=19, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=2, item=2
<=  2: processing received queued item, id=20, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=5, item=5
** completed: worker-id=7, item=14
** completed: worker-id=8, item=7
<=  5: processing received queued item, id=21, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  7: processing received queued item, id=22, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  8: processing received queued item, id=23, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=9, item=10
** completed: worker-id=4, item=17
<=  9: processing received queued item, id=24, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=3, item=12
<=  4: processing received queued item, id=25, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  3: processing received queued item, id=26, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=12, item=19
<= 12: processing received queued item, id=27, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=2, item=20
** completed: worker-id=1, item=13
** completed: worker-id=6, item=15
** completed: worker-id=5, item=21
<=  1: processing received queued item, id=28, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  2: processing received queued item, id=29, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<=  6: processing received queued item, id=30, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=11, item=16
<=  5: processing received queued item, id=31, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
<= 11: processing received queued item, id=32, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=10, item=18
<= 10: processing received queued item, id=33, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=4, item=25
<=  4: processing received queued item, id=34, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=12, item=27
<= 12: processing received queued item, id=35, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=7, item=22
** completed: worker-id=2, item=29
** completed: worker-id=8, item=23
** completed: worker-id=9, item=24
** completed: worker-id=3, item=26
** completed: worker-id=10, item=33
** completed: worker-id=4, item=34
** completed: worker-id=5, item=31
<=  2: processing received queued item, id=36, item=2024-09-18_13:08:45:N_PDT_1726690125 ...
** completed: worker-id=1, item=28
** completed: worker-id=6, item=30
** completed: worker-id=11, item=32
** completed: worker-id=12, item=35
** completed: worker-id=2, item=36

Cleaning up coordination files ...
```

Inspired by:
* https://hackthology.com/a-job-queue-in-bash.html (Tim Henderson)
  * https://gist.github.com/timtadh/c974db5390457228189bba611bf8fe64
* https://serverfault.com/questions/347582/adding-a-random-delay-for-a-linux-command
</details>

<details>
  <summary>progess bar w/percentage</summary>

Demonstrates giving feed back via counters, `%` percentages and progress bars.

Usage:
```
bin/progress
```
```
Completed 1 item, progress: ( 1/25) [#                        ]   4% ...
Completed 1 item, progress: ( 2/25) [##                       ]   8% ...
Completed 1 item, progress: ( 3/25) [###                      ]  12% ...
Completed 1 item, progress: ( 4/25) [####                     ]  16% ...
Completed 1 item, progress: ( 5/25) [#####                    ]  20% ...
Completed 1 item, progress: ( 6/25) [######                   ]  24% ...
Completed 1 item, progress: ( 7/25) [#######                  ]  28% ...
Completed 1 item, progress: ( 8/25) [########                 ]  32% ...
Completed 1 item, progress: ( 9/25) [#########                ]  36% ...
Completed 1 item, progress: (10/25) [##########               ]  40% ...
Completed 1 item, progress: (11/25) [###########              ]  44% ...
Completed 1 item, progress: (12/25) [############             ]  48% ...
Completed 1 item, progress: (13/25) [#############            ]  52% ...
Completed 1 item, progress: (14/25) [##############           ]  56% ...
Completed 1 item, progress: (15/25) [###############          ]  60% ...
Completed 1 item, progress: (16/25) [################         ]  64% ...
Completed 1 item, progress: (17/25) [#################        ]  68% ...
Completed 1 item, progress: (18/25) [##################       ]  72% ...
Completed 1 item, progress: (19/25) [###################      ]  76% ...
Completed 1 item, progress: (20/25) [####################     ]  80% ...
Completed 1 item, progress: (21/25) [#####################    ]  84% ...
Completed 1 item, progress: (22/25) [######################   ]  88% ...
Completed 1 item, progress: (23/25) [#######################  ]  92% ...
Completed 1 item, progress: (24/25) [######################## ]  96% ...
Completed 1 item, progress: (25/25) [#########################] 100% ...
```

Inspired by:
* https://stackoverflow.com/questions/12498304/using-bash-to-display-a-progress-indicator-spinner
</details>

<details>
  <summary>spinner</summary>

Rewrites the "active" line in the terminal.

Usage:
```
bin/spinner
```
```
5(s) [-] ...
Completed in 5(s)
```
[![asciicast](https://asciinema.org/a/RwAd8F456OqiWYX46mldIpWaL.svg)](https://asciinema.org/a/RwAd8F456OqiWYX46mldIpWaL)

Inspired by:
* https://stackoverflow.com/questions/12498304/using-bash-to-display-a-progress-indicator-spinner
</details>

<details>
  <summary>resumable state files</summary>

Reading/Writing file based processing state for the purpose of 
marking failed items and supporting stopping/resuming.

Usage:
```
bin/resume-files
```
```
=> sleep 1
- [x] echo ...
- [!] /usr/bin/false
- [x] sleep 1
- [ ] echo ...
- [ ] false
- [ ] host www.google.com
=> echo ...
...
- [x] echo ...
- [!] /usr/bin/false
- [x] sleep 1
- [x] echo ...
- [ ] false
- [ ] host www.google.com
=> false
- [x] echo ...
- [!] /usr/bin/false
- [x] sleep 1
- [x] echo ...
- [!] false
- [ ] host www.google.com
=> host www.google.com
www.google.com has address 142.250.189.164
www.google.com has IPv6 address 2607:f8b0:4005:810::2004
- [x] echo ...
- [!] /usr/bin/false
- [x] sleep 1
- [x] echo ...
- [!] false
- [x] host www.google.com
All tasks were attempted/completed.

Cleaning up state file ...
```

Inspired by:
* https://maximerobeyns.com/fragments/job_queue (Maxime Robeyns)
</details>

<details>
  <summary>log stdout, stderr to a file while preserving terminal output</summary>

Saves the script output to file, removing color codes on exit.
Preserves terminal output w/color.

Usage:
```
bin/output-logs
```
```
Log file: output-YYYY-mm-dd_HH-MM-SS.log
total 64
-rw-r--r--@  1 <user>  staff    119 Sep 17 17:54 Brewfile
-rw-r--r--   1 <user>  staff   6909 Sep 17 17:49 Brewfile.lock.json
-rw-r--r--   1 <user>  staff   1074 Sep 17 13:37 LICENSE
-rw-r--r--@  1 <user>  staff  15878 Sep 18 13:47 README.md
drwxr-xr-x@ 10 <user>  staff    320 Sep 19 14:02 bin <blue>
stderr -> ...
console only stdout
console only stderr
```

Inspired by:
* https://stackoverflow.com/questions/17998978/removing-colors-from-output
  * https://serverfault.com/questions/103501/how-can-i-fully-log-all-bash-scripts-actions
</details>

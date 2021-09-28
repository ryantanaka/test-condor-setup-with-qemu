# HTCondor Container Setup

This contains an example HTCondor setup that uses 4 Docker containers, each 
playing one of the 3 HTCondor roles. The 4 containers are as follows:

1. condor9 central manager (x86_64)
2. condor9 submit (x86_64) 
3. condor9 execute (x86_64)
4. condor8.8 execute (aarch64) <-- requires qemu

The following example will illustrate 2 issues encountered when using qemu:
1. procd had to be disabled on aarch64 execute node
2. file transfers don't work between aarch64 execute node and x86_64 submit node

## Files
```
├── condor8.8-arm64-worker     
│   ├── 10-htcondor.conf    <--- contains configs
│   ├── 50-test-setup.conf  <--- contains configs
│   ├── condor-packages
│   │   ├── htcondor_8.8.13-1+deb10u0_arm64.deb
│   │   ├── libclassad14_8.8.13-1+deb10u0_arm64.deb
│   │   └── libclassad-dev_8.8.13-1+deb10u0_arm64.deb
│   ├── Dockerfile
│   └── supervisord.conf
├── condor9.1               <--- Dockerfiles for condor9 containers (official HTCondor containers + some configs)
│   ├── central-manager
│   │   └── Dockerfile
│   ├── execute
│   │   └── Dockerfile
│   └── submit
│       └── Dockerfile
├── docker-compose.yml  
└── README.md
```

## Requirements
- Docker
- Docker Compose
- QEMU support: https://github.com/multiarch/qemu-user-static
    - When running this, if you see `sh: write error: Invalid argument`, see
        https://github.com/multiarch/qemu-user-static/issues/100#issuecomment-566083588

## Usage

1. `git clone https://github.com/ryantanaka/test-condor-setup-with-qemu`
2. `cd condor9-x86_64-test-pool`
3. `docker-compose up` <-- use `-d` to run in background
4. In another terminal, run `docker exec -it submit /bin/bash`. This will bring you
    into the submit container. Running `condor_status` should show both the
    aarch64 and x86_64 machines as expected.   

```
Name               OpSys      Arch    State     Activity LoadAv Mem    ActvtyTime

slot1@1cb2f0c1297a LINUX      X86_64  Unclaimed Idle      0.000  1024  0+00:14:28
slot1@ace26a7179c0 LINUX      aarch64 Unclaimed Idle      0.000 31720  0+00:17:49

               Total Owner Claimed Unclaimed Matched Preempting Backfill  Drain

  X86_64/LINUX     1     0       0         1       0          0        0      0
 aarch64/LINUX     1     0       0         1       0          0        0      0

         Total     2     0       0         2       0          0        0      0
```
5. Change the `submituser` with `su - submituser` and `cd /home/submituser/demo`.
6. Run `condor_submit job.sub` to submit a shell script which will run `cat /etc/os-release`
    on the x86_64 execute node. This should succeed.
7. Add `requirements = Arch == "aarch64"` to `job.sub`. Run `condor_submit job.sub`.
    This should fail with the following log:

```
[submituser@9f73eb9c4e29 demo]$ cat job.log
000 (003.000.000) 2021-09-28 00:10:51 Job submitted from host: <172.25.0.5:9618?addrs=172.25.0.5-9618&alias=9f73eb9c4e29&noUDP&sock=schedd_26_698d>
...
040 (003.000.000) 2021-09-28 00:11:03 Finished transferring input files
...
007 (003.000.000) 2021-09-28 00:11:03 Shadow exception!
        Error from slot1_1@ace26a7179c0: Failed to transfer files
        0  -  Run Bytes Sent By Job
        0  -  Run Bytes Received By Job
...
```
## Notes
1. The aarch64 execute node is configured with the following:
```
# DEBUGGING
STARTER_DEBUG = D_ALL:2
CREATE_CORE_FILES = TRUE
ABORT_ON_EXCEPTION = TRUE
NOT_RESPONDING_WANT_CORE = TRUE
```

and after running the aarch64 job, `StarterLog.slot1_1` shows the following error:

```
./StarterLog.slot1_1:09/28/21 00:31:03 (fd:14) (pid:524) (D_ALWAYS|D_FAILURE) ERROR "Failed to transfer files" at line 2468 in file /build/condor-8.8.13/src/condor_starter.V6.1/jic_shadow.cpp
```

2. `USE_PROCD = FALSE` must be set for the aarch64 execute node or else it won't start up.
3. The **two errors above only happen when using qemu** for the aarch64 execute node. When using
    the same container on a native aarch64 devices like a raspberry pi, there were
    no issues encountered. 
 







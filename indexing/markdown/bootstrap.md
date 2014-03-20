##System Bootstrap Sequence


System bootstrap will be initiated by ns_server. It will bring up all the processes with required parameters.
Once a process is up, it follows its own bootstrap sequence and then either waits for further commands to be given 
or contacts Index Coordinator to get the latest StateContext and takes further actions based on that.

ns_server also watchdogs each process and restarts it in case of failure/crash.

In case a node goes down, ns_server will detect that and pass this information to Index Coordinator Master.
If Index Coordinator Master node has gone down, ns_server will elect a new master.


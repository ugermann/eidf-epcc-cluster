## Addressing Resource Shortages on Login Node - Priority 3

The login node has experienced shortages in CPU and disk space multiple times. Although upgrades have been made to alleviate these issues, I propose keeping this topic open for further improvements.

### Login Node Resource Shortages:

#### Summary:
The login node has already run out of CPU and disk space on separate occasions. Additional CPU cores and larger disk space have been allocated, but further solutions are required to prevent future occurrences.

#### Existing Ideas:

- **Resource Monitoring Alarms**: Implement alarms that trigger when CPU, disk, RAM, or network bandwidth reach dangerous levels of utilization on the login node. Immediate notifications to the support team would enable quick action to prevent user interruption.

- **Increase RAM and CPU Cores**: The current 4-core setup is insufficient, especially when a user runs an intensive task by mistake, affecting the entire cluster. Adding more cores would not only provide a buffer against such incidents but also improve the efficiency of environment setup and job script launching. Even with a CPU scheduler distributing time among users, additional cores would offer significant advantages.

Keeping a close eye on the login node's resources will improve the overall stability and usability of the cluster.


## Update: 07/11/2023
The head node has been upgraded with 8 CPUs (from 4), and with 16GB of RAM from (8GB), furthermore, a number of OS-level scheduling processes have been put in place to ensure a more fair CPU-time allocation to users. 
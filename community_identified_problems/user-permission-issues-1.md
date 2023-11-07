## Permissions and Configuration Issues with the University of Edinburgh's Kubernetes GPU Cluster - Priority 1

#### Summary
The issues we're facing relate both to user permissions and cluster command restrictions. 

1) User Permissions: Presently, it appears that users possess the ability to terminate each other's pods and jobs, which can lead to premature job termination and loss of computational progress. This is a critical issue affecting collaboration and respectful shared use of our computational resources.

2) Command Execution: Essential commands such as `kubectl get nodes` are currently inaccessible. This limits our ability to effectively monitor and troubleshoot our workloads, and impacts the efficiency and transparency of resource use and allocation.

3) Misconfigured Permissions: Overall, the permissions settings across our Kubernetes GPU cluster seem to be incorrectly configured. This is resulting in erratic and unreliable behaviour, impeding our data processing and analysis capabilities. 

#### Proposed Solutions

1) Isolate User Permissions: Implement a namespace for each user or each team. Namespaces are a way to divide cluster resources between multiple users. This can shield active jobs and pods from unintended interference, and thereby safeguard the progress of ongoing computational tasks.

2) Grant Essential Command Access: Reevaluate command permissions and provide access to essential troubleshooting and monitoring commands like `kubectl get nodes`. This will empower our users to independently monitor their jobs and proactively address computational challenges.

3) Review and Correct Permission Configuration: Conduct a systematic review and correction of current permission configurations across our Kubernetes GPU cluster. This action can potentially resolve the inherent irregularities interrupting our daily operations.

I believe these changes could drastically improve our computational and collaborative capabilities, and I'm looking forward to your assistance in implementing them. 

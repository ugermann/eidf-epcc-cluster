# eidf-epcc-cluster

Additional documentation from the team managing the EIDF Cluster is available at [this link](https://git.ecdf.ed.ac.uk/gpu_cluster_support/eidf.gpu.cluster.docs). If you don't have access to that repository, you can use [this link](https://epcced.github.io/eidf-docs/services/gpuservice/). Likewise, if you need to flag problems with the cluster, please get in touch with the helpdesk at [this link](https://www.epcc.ed.ac.uk/contact).

About running ML/NLP experiments on a Kubernetes cluster -- we prepared an introductory guide available [here](https://github.com/uoe-eidf-cluster-users/eidf-epcc-cluster/blob/main/KUBERNETES-for-ML.md) -- if you find that anything is missing from that guide, please feel free to add to it (you all have write access) or, if you are unable of doing so, please open an issue.

We now also have a Slack channel for the EIDF cluster! You can join by following [this link](https://join.slack.com/t/slack-czc7871/shared_invite/zt-1wmdgxikm-KTGKTHF2Ez1s0zS0lziF9g) --  From experience we know that with shared resources, it's often easy to resolve any issues before they even occur by just being aware of each other and being able to communicate any issues that we are having (which can include jobs being blocked by another persons jobs etc). Or altogether to be used by new users, or existing users trying to do something.

- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **WARNING** the cluster does not have any quota or permission management at the moment, so please **behave** and don't hoard resources or make life harder for other users, or we will have to restrict your access.

# EIDF EPCC Cluster -- Guide for new users

A guide to onboarding new users. Be aware that this is a developing service.

## New User Sign Up to EIDF

Full Documentation on signing up at [EIDF Documentation](https://epcced.github.io/eidf-docs/).

1. Open Browser and goto the [EIDF Portal](https://portal.eidf.ac.uk)
2. Click Login -> This will redirect to SAFE
3. if you have a SAFE account -> use that account, if you do not have a SAFE account, register for a new SAFE account
4. Return to the EIDF portal after your SAFE account has logged in and activated.

## New User Request Access to Informatics Project

1. Click on the dropdown menu "Projects" 
2. Click "Request Access"
3. Choose from the list "eidf029 - Informatics K8s Support"
4. Click "Apply'
5. An approver will add you to the project and create an account.

## Approver Steps to Create EIDF Accounts for New User

1. Login into the [EIDF Portal](https://portal.eidf.ac.uk)
2. Click on the dropdown menu "Projects"
3. Choose the applicant to the project from the "Project Management Requests"
4. Choose the approve option in the page and submit.
5. Click on the dropdown menu "Projects"
6. Click "Your Projects"
7. Choose from the list "eidf029 - Informatics K8s Support"
8. Find and click the button for "Create Account"
9. Create a username for the account in the form <initial><surname>-infk8s e.g dmckay-infk8s
10. In the account owner drop down box, choose the applicant you are creating the account for.
11. Click "Submit"
12. In the "Project Accounts", click "Manage" next to the account you have just created.
13. Give "Access", not "Sudo" to the following entries: eidf-gateway, eidf029-host1

## New User First Login

1. Login into the [EIDF Portal](https://portal.eidf.ac.uk)
2. Click on the dropdown menu "Projects"
3. Click "Your Projects"
4. Choose from the list "eidf029 - Informatics K8s Support"
5. Click your account user name from your Account
6. Click to view your initial password and copy/note it
7. Upload your SSH public key to "Credentials" 
8. Click VDI Login
9. From the project list of VMs, choose eidf029-host1_ssh
10. Enter your username
11. Enter the initial password
12. At the change prompt follow the instructions.

## New User Login

1. Use the VDI as on initial login, save for changing password
2. Optional use the EIDF Gateway
```
ssh -J USERNAME@eidf-gateway.epcc.ed.ac.uk USERNAME@10.24.5.121
```
Alternatively, you can edit the `.ssh/config` file (useful for VSCode)
```
Host eidf
    User USERNAME
    IdentityFile PATH-TO-KEY
    HostName 10.24.5.121
    ProxyJump USERNAME@eidf-gateway.epcc.ed.ac.uk
```
and access the cluster by `ssh eidf`.

You can run `kubectl get pods` to check that the kubeconfig is recognised.

## Downloading Kubernetes on the Cluster (If Needed)
Steps 1-5 are based on the official Kubernetes website: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

1. Download the Kubernetes command-line tool `kubectl` using the following command:
```
curl -LO https://dl.k8s.io/release/v1.29.1/bin/linux/arm64/kubectl
```

2. To validate the binary:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

3. To verify that the downloaded binary matches the checksum:
```
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```
If the binary is valid, the output will display: `kubectl: OK`

4. Install Kubernetes:
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

5. To confirm that kubectl is installed correctly and is up to date, run:
```
kubectl version --client
```

6. Set up Kubernetes configuration:
```
export KUBECONFIG=/kubernetes/config
```

7. When running kubectl commands, specify the namespace with the --namespace flag:
```
[command] --namespace eidf097ns
```
Replace [command] with the specific kubectl command you wish to run.

8. Alternatively, create an alias for convenience:
```
k = "kubectl --namespace eidf097ns"
```
For steps 7-8, Remember to replace eidf097ns with the actual namespace you are working with.

## Run a pod

1. open editor of your choice to create the file test_NBody.yml
2. put the following into to the file:

```yaml
 apiVersion: v1
 kind: Pod
 metadata:
   generateName: sample-
 spec:
   restartPolicy: Never
   containers:
   - name: cudasample
     image: nvcr.io/nvidia/k8s/cuda-sample:nbody-cuda11.7.1
     args: ["-benchmark", "-numbodies=512000", "-fp64", "-fullscreen"]
     resources:
       limits:
          nvidia.com/gpu: 1
```

3. Save the file and exit the editor
4. Run `kubectl create -f test_NBody.yml -l eidf/user=$(whoami)`

   As of February 2024 (this is an requirement specific to the School of Informatics), you must provide the label `eidf/user` pointing to your user name. You can do this via `-l eidf/user=$(whoami)` as above, or put this into your `*.yml` file in the metadata section like this:
   ```
   metadata:
     generateName: sample-
     labels:
        eidf/user: <your user name on the EIDF head node>
   ```
5. This will output something like:

```bash
pod/sample-7gdtb created
```
6. Run `kubectl get pods`
7. This will output something like:

```bash
pi-tt9kq                                                          0/1     Completed   0              24h
sample-24n7n                                                      0/1     Completed   0              24h
sample-2j5tc                                                      0/1     Completed   0              24h
sample-2kjbx                                                      0/1     Completed   0              24h
sample-2mnvg                                                      0/1     Completed   0              24h
sample-4sng2                                                      0/1     Completed   0              24h
sample-5h6sr                                                      0/1     Completed   0              24h
sample-6bqql                                                      0/1     Completed   0              24h
sample-7gdtb                                                      0/1     Completed   0              39s
sample-8dnht                                                      0/1     Completed   0              24h
sample-8pxz4                                                      0/1     Completed   0              24h
sample-bphjx                                                      0/1     Completed   0              24h
sample-cp97f                                                      0/1     Completed   0              24h
sample-gcbbb                                                      0/1     Completed   0              24h
sample-hdlrr                                                      0/1     Completed   0              24h
sample-hkwk2                                                      0/1     Completed   0              24h
sample-j66ck                                                      0/1     Completed   0              24h
sample-jxhtk                                                      0/1     Completed   0              24h
sample-lzmg8                                                      0/1     Completed   0              24h
sample-nhrtk                                                      0/1     Completed   0              24h
sample-rh9v7                                                      0/1     Completed   0              24h
sample-v48jd                                                      0/1     Completed   0              24h
```

8. View the logs of the pod you ran `kubectl logs sample-7gdtb`
9. This will output something like:

```bash
Run "nbody -benchmark [-numbodies=<numBodies>]" to measure performance.
	-fullscreen       (run n-body simulation in fullscreen mode)
	-fp64             (use double precision floating point values for simulation)
	-hostmem          (stores simulation data in host memory)
	-benchmark        (run benchmark to measure performance) 
	-numbodies=<N>    (number of bodies (>= 1) to run in simulation) 
	-device=<d>       (where d=0,1,2.... for the CUDA device to use)
	-numdevices=<i>   (where i=(number of CUDA devices > 0) to use for simulation)
	-compare          (compares simulation results running once on the default GPU and once on the CPU)
	-cpu              (run n-body simulation on the CPU)
	-tipsy=<file.bin> (load a tipsy model file for simulation)

NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.

> Fullscreen mode
> Simulation data stored in video memory
> Double precision floating point simulation
> 1 Devices used for simulation
GPU Device 0: "Ampere" with compute capability 8.0

> Compute 8.0 CUDA device: [NVIDIA A100-SXM4-40GB]
number of bodies = 512000
512000 bodies, total time for 10 iterations: 10570.778 ms
= 247.989 billion interactions per second
= 7439.679 double-precision GFLOP/s at 30 flops per interaction
```
10. delete your pod with `kubectl delete pod sample-7gdtb`

## Kubectl Secret Management

For many jobs it may be necessary to leverage kubectl secrets (e.g. to pull a private image). Each user account has access to create their own secrets directly via the CLI using `kubectl create secret`. In order to help maintain secrets associated to the cluster, we ask that users preface their secrets with a unique identifier (e.g. student number). For instance a secret for authenticating into a private GitHub container registry might follow the template <unique_id>-ghcr-secret which could look like s1234567-ghcr-secret (note: in this case the word secret is superfluous in the secret name and could be dropped). 

## Running your own experiments
	
Follow [this](https://github.com/AntreasAntoniou/minimal-ml-template/tree/main/kubernetes) guide to get started, and check the following tools from the amazing @AntreasAntoniou:

- https://github.com/BayesWatch/kubeproject for general kubectl stuff and understanding what’s going on.
- https://github.com/AntreasAntoniou/kubejobs for python-based kubernetes job launching that covers a lot of options for the yaml — but in python class format.
- https://github.com/AntreasAntoniou/minimal-ml-template/tree/main/kubernetes for a minimal ml projects that can run on a kubernetes cluster

## Acknowledging EIDF in your work

More details [here](https://epcced.github.io/eidf-docs/overview/acknowledgements/)

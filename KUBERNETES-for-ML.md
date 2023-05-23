# Kubernetes for ML/NLP Experiments

![image](https://github.com/EdinburghNLP/eidf-epcc-cluster/assets/227357/60cc6f3e-6bf2-4508-9b74-05ddadc028c0)

If you’re new to Kubernetes, you can think of it as the operating system that runs on your cluster.

Just like how the operating system abstracts away the various hardware resources, Kubernetes does the same with a cluster and abstracts away details about the actually infrastructure setup. This offers users a consistent runtime and development environment for a cluster, similar to locally-hosted operating systems. Data scientists and developers don’t have to worry about infrastructure-related issues. They can focus on their apps and let Kubernetes manage the infrastructure.

Before we jump into how we can use Kubernetes to run large-scale AI experiments, let’s introduce a few key concepts you should know as you follow along with the example.

## Containers, Pods, and Jobs

The basic deployable unit in Kubernetes is called a Pod. A Pod contains one or more processes in co-located containers which share the same volumes, IP address, and namespace. A pod never spans multiple nodes. For a training workload, a Pod may only consist of one container running your favorite deep learning framework as illustrated in thw following figure.

![image](https://github.com/EdinburghNLP/eidf-epcc-cluster/assets/227357/1bdeaa3f-fa03-4747-930c-127c14f599c3)

A Pod is an unmanaged resource in Kubernetes. This means if a failure occurs, the pod simply ceases to exist. Therefore, we can introduce a resource manager called Job, which is responsible for automatically rescheduling failed pods onto different nodes. Job resources are useful for workloads that go to completion like a training job.

A Pod can also have multiple containers to run processes like data pre- and post-processing for inference deployments. Inference workloads typically run as highly available services, running as continuous tasks that are never considered complete. A different resource manager called ReplicaSet can make sure that specified replicas of this job are always available and running forever unless terminated by the user.

Kubernetes boils down to one key idea: infrastructure abstraction. A user should never have to think in terms of servers and nodes. They should focus on specifying jobs and containers in a pod, and Kubernetes takes care of all the underlying infrastructure management.

Now that we’ve briefly introduced Pods and Job resources, Let’s now dive into our example.

## Hyperparameters Optimization for Training a Model on the MNIST Dataset

In ML, hyperparameters typically include options such as learning rate schedule, batch size, data augmentation options and others. Each option greatly affects the model accuracy on the same dataset. Two of the most common strategies for selecting the best hyperparameters for a model are grid search and random search. In the grid search method  (also known as the parameter sweep method) you define the search space by enumerating all possible hyperparameter values and train a model on each set of values. Random search only select random sets of values sampled from the exhaustive set. The results of each training run are then validated against a separate validation set.

When starting from scratch, random search and grid search are still the best approaches to narrow down the search space. The challenge with grid search and random search is that introducing more than a few hyperparameters and options quickly results in a combinatorial explosion. Add to this the training time required for each hyperparameter set and the problem quickly becomes intractable. Thankfully, this is also an embarrassingly parallel problem since each training run can be performed independently of others.

This is where Kubernetes comes in.

In this example, I present a general framework for running large-scale hyperparameter search experiments using Kubernetes on a GPU cluster. The framework is flexible and allows you to do grid search or random search and implements "version everything" so you can trace back all previously run experiments.

Assuming you’ve already started by setting up a Kubernetes cluster, our solution for running hyperparameter search experiments consists of the following 7 steps:

- Specify hyperparameter search space;
- Develop a training script that can accept hyperparameters and apply them to the training routine;
- Push training scripts and hyperparameters in a Git repository for tracking;
- Specify Kubernetes Job specification files with application container to run and GPU resources required;
- Submit multiple Kubernetes job requests (one per hyperparameter set) using above specification template;
- Analyze the results and pick the hyperparameter set.

## Step 1: Specify the Hyperparam Search Space

Open the file `generate-grid-cli.py` and update the `hyper_params` variable to specify the hyperparameter space that you wish to cover. You can specify specific values, ranges or samples from specific probability distributions.

```python
hyper_params = {
    'batch_size': [32, 64, 128, 512],
}
```

After specifying the hyperparameters run the following script.

```bash
> python generate-grid-cli.py
Total number of hyperparameter sets: 4
Hyperparameter sets saved to: hyperparams.yml
```

This should generate a YAML file called `hyperparams.yml`, a plain text file which stores all the hyperparameter sets. This is the most important file in this example since each Kubernetes Pod will read this file and pick one hyperparameter set to run training on as illustrated in the following figure.

![image](https://github.com/EdinburghNLP/eidf-epcc-cluster/assets/227357/cf2de83d-ae0e-4b3d-97d6-fedf5185c607)

## Step 2: Develop a Training Script

Now let's train a neural network on the MNIST dataset. When the training script executes on a Kubernetes worker node, it queries its unique job_id from an environment variable called JOB_ID which is unique to each Kubernetes Job. It then reads hyperparams.yml and returns the hyperparameter set corresponding the job id: `hyper_param_set[job_id-1]["hyperparam_set"]`

```python
def get_hyperparameters(job_id):
   with open("hyperparams.yml", 'r') as stream:
      hyper_param_set = yaml.load(stream)
   return hyper_param_set[job_id-1]["hyperparam_set"]
```

## Step 3: Push training scripts and hyperparameters in a Git repository for tracking

In order to make the hyperparameters sets inhyperparams.yml you generated in Step 2 available to all Pods once you submit jobs, you need to push the changes back to your git repository. You may be asked to provide your GitHub login details.

```bash
> git add hyperparams.yml
> git commit -m "Updated hyperparameters. Includes 4 hyperparameter sets"
> git push
```

## Step 4: Specify the Kubernetes Job specification files in YAML

We're now ready to tell Kubernetes to run our hyperparameter sweep experiment. Kubernetes makes it very easy to spin up resources using configuration files written in YAML. Kubernetes supports both YAML and JSON, but YAML tends to be much friendlier to read and write. If you're not familiar with it, you'll see how easy it is to use.

Navigate to `~/eidf-kubernetes-example/kubernetes-hyperparam-exp` and open `mnist-job-template.yml`; let's first take a look at the key parts of the template yaml spec file:

```yaml
containers:
  - name: pytorch
    image: nvcr.io/nvidia/pytorch:22.01-py3
    workingDir: /mnist-training
    env:
    - name: JOB_ID
      value: "$ITEM"
    command: ["bash"]
    args: ["-c","python main.py"]
    computeResourceRequests: ["nvidia-gpu"]
```

In this example, each job runs a single container. We specify that Kubernetes needs to pull the PyTorch container from the NVIDIA GPU Cloud (NGC) container registry. Next we specify an environment variable called `JOB_ID`, with a value currently called `$ITEM`. This placeholder will get replaced by a unique number for each Job.

Command and `args` tells Kubernetes which command to run inside the container. Notice that every container runs the same `main.py` script. So how does each worker know what hyperparameter set to choose and run?

This is where the environment variable `$JOB_ID` comes in. Kubernetes introduces an environment variable in each container and  `main.py` will read the environment variable and pick the corresponding hyperparameter set from `hyperparams.yml`.

```yaml
volumes:
- name: mnist-training
  gitRepo:
    repository: https://github.com/pminervini/eidf-kubernetes-example.git
    revision: master
    directory: .
```

When Kubernetes starts a container on a worker node, it starts of with the exact same files that were added to the container image during build time. Since we’re pulling a PyTorch container image from NGC, the container only contains PyTorch and it’s supporting libraries. Kubernetes volumes allow us to mount a temporary or external storage volume in the container, and make it available to your application. In the snippet above, we specify one type of volume:

- **gitRepo**: This instructs Kubernetes to initialize a local volume and checkout the contents specified in the `repository` URL. Be sure to update the URL to match your personal GitHub repo that contains your changes.

```yaml
computeResources:
  - name: "nvidia-gpu"
  resources:
    limits:
      nvidia.com/gpu: 1
```

Finally, we specify the compute resources for each training run. In this example, we use only one GPU per training run using one set of hyperparameters. If the training code supports multi-GPU training and you have more than one GPU available, then you can increase the number of GPUs here. Kubernetes will automatically find a node with the requested number of GPUs and schedule the job there.

## Step 5: Submit Multiple Kubernetes Job Requests

In our example, we run multiple Kubernetes Jobs – one job per hyperparameter set – each with a unique identifier, so that no two jobs run on the same hyperparameter set specified in `hyperparams.yml` we created in Step 1.

To achieve this, we’ll use the [Parallel Processing using Expansions](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/) approach described in the Kubernetes documentation. The idea is that you create a template (`mnist-job-template.yml`) that describes the job that we need to run.

```yaml
env:
  - name: JOB_ID
    value: "$ITEM"
```

In the template is a unique placeholder called `$ITEM`. Running the `create_jobs.sh` script (as shown below) expands the template into multiple Job specification files, one for each hyperparameter set to be trained on. `$ITEM` in each generated Job specification file will replaced by a unique Job number.

```bash
> cd ~/workspace/eidf-kubernetes-example 
> ./create-jobs.sh 4
> ls hyperparam-jobs-specs 
README.md	mnist-job-1.yml	mnist-job-2.yml	mnist-job-3.yml	mnist-job-4.yml
```

The above code snippet generates 4 separate job specification files. Each file has a unique name and a unique environment variable that enables each worker to query a hyperparameter set based on its `$JOB_ID`.

To submit a job on the EIDF cluster, just run:

```bash
> kubectl create -f hyperparam-jobs-specs/
job.batch "mnist-single-job-1" created
job.batch "mnist-single-job-2" created
job.batch "mnist-single-job-3" created
[..]
```

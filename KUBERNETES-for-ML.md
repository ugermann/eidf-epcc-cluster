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

## Hyperparameters Optimization for Training a Model on the MNIST Dataset

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


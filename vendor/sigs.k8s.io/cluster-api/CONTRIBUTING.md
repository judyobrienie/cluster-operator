# Contributing Guidelines

Read the following guide if you're interested in contributing to cluster-api.

## Contributor License Agreements

We'd love to accept your patches! Before we can take them, we have to jump a couple of legal hurdles.

Please fill out either the individual or corporate Contributor License Agreement (CLA). More information about the CLA and instructions for signing it [can be found here](https://github.com/kubernetes/community/blob/master/CLA.md).

***NOTE***: Only original source code from you and other people that have signed the CLA can be accepted into the repository.

## Finding Things That Need Help

If you're new to the project and want to help, but don't know where to start, we have a semi-curated list of issues that should not need deep knowledge of the system. [Have a look and see if anything sounds interesting](https://github.com/kubernetes-sigs/cluster-api/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22). Alternatively, read some of the docs on other controllers and try to write your own, file and fix any/all issues that come up, including gaps in documentation!

## Contributing a Patch

1. If you haven't already done so, sign a Contributor License Agreement (see details above).
1. Fork the desired repo, develop and test your code changes. 
1. Submit a pull request.

All changes must be code reviewed. Coding conventions and standards are explained in the official [developer docs](https://github.com/kubernetes/community/tree/master/contributors/devel). Expect reviewers to request that you avoid common [go style mistakes](https://github.com/golang/go/wiki/CodeReviewComments) in your PRs.

### Merge Approval

Cluster API maintainers may add "LGTM" (Looks Good To Me) or an equivalent comment to indicate that a PR is acceptable. Any change requires at least one LGTM.  No pull requests can be merged until at least one Cluster API maintainer signs off with an LGTM.

## Cloud Provider Dev Guide

### Overview

The Cluster API is a Kubernetes project to bring declarative, Kubernetes-style APIs to cluster creation, configuration, and management. It provides optional, additive functionality on top of core Kubernetes.

This document is meant to help OSS contributors implement support for providers (cloud or on-prem).

As part of adding support for a provider (cloud or on-prem), you will need to:

1.  Create tooling that conforms to the Cluster API (described further below)
1.  A machine controller that can run independent of the cluster. This controller should handle the lifecycle of the machines, whether it's run in-cluster or out-cluster.

The machine controller should be able to act on a subset of machines that form a cluster (for example using a label selector).

### Resources

*   [Cluster Management API KEP](https://github.com/kubernetes/community/blob/master/keps/sig-cluster-lifecycle/0003-cluster-api.md)
*   [Cluster type](https://github.com/kubernetes-sigs/cluster-api/blob/master/pkg/apis/cluster/v1alpha1/cluster_types.go#L40)
*   [Machine type](https://github.com/kubernetes-sigs/cluster-api/blob/master/pkg/apis/cluster/v1alpha1/machine_types.go#L42)

### Boostrapping

To minimize code duplication and maximize flexibility, bootstrap clusters with an external Cluster Management API Stack. A Cluster Management API Stack contains all the components needed to provide Kubernetes Cluster Management API for a cluster. [Bootstrap Process Design Details](https://docs.google.com/document/d/1CnzIXtitfbO6Y7ZxVWROGO8jr19t0vooDx-YQ7c2nbI/edit?usp=sharing).

### A new Machine can be created in a declarative way

**A new Machine can be created in a declarative way, including Kubernetes version and container runtime version. It should also be able to specify provider-specific information such as OS image, instance type, disk configuration, etc., though this will not be portable.**

When a cluster is first created with a cluster config file, there is no master node or api server. So the user will need to bootstrap a cluster. While the implementation details are specific to the provider, the following guidance should help you:

* Your tool should spin up the external apiserver and the machine controller.
* POST the objects to the apiserver.
* The machine controller creates resources (Machines etc)
* Pivot the apiserver and the machine controller in to the cluster.

### A specific Machine can be deleted, freeing external resources associated with it.

When the client deletes a Machine object, your controller's reconciler should trigger the deletion of the Machine that backs that machine. The delete is provider specific, but usually requires deleting the VM and freeing up any external resources (like IP).

### A specific Machine can be upgraded or downgraded

These include:

*   A specific Machine can have its kubelet version upgraded or downgraded.
*   A specific Machine can have its container runtime changed, or its version upgraded or downgraded.
*   A specific Machine can have its OS image upgraded or downgraded.

A sample implementation for an upgrader is [provided here](https://github.com/kubernetes-sigs/cluster-api/blob/master/tools/upgrader/util/upgrade.go). Each machine is upgraded serially, which can amount to:

```
for machine in machines:
    upgrade machine
```

The specific upgrade logic will be implement as part of the machine controller, and is specific to the provider. The user provided provider config will be in `machine.Spec.ProviderConfig`.

Discussion around in-place vs replace upgrades [is here](https://github.com/kubernetes/community/blob/master/keps/sig-cluster-lifecycle/0003-cluster-api.md#in-place-vs-replace).

## Support Channels

Whether you are a user or contributor, official support channels include:

- GitHub issues: https://github.com/kubernetes-sigs/cluster-api/issues/new
- Slack: Chat with us on [Slack](http://slack.k8s.io/): #cluster-api
- Email: [kubernetes-sig-cluster-lifecycle](https://groups.google.com/forum/#!forum/kubernetes-sig-cluster-lifecycle) mailing list

Before opening a new issue or submitting a new pull request, it's helpful to search the project - it's likely that another user has already reported the issue you're facing, or it's a known issue that we're already aware of.
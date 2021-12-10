# Fleet Rancher Logging Demo
This repository aims to provide some baseline to deploy logging in a downstream cluster using Fleet, it is not meant to be a reference to logging. It is just a test for Fleet bundle dependencies.

## Structure
This Repo can be referenced in Fleet as-if, by providing the paths: eck, cluster-output, rancher-logging and rancher-logging-crd.

## Pre-requisites
Very important to mention at this stage, bundle dependency in Fleet 0.3.6-0.3.7 is still a bit problematic in terms of UX, mainly:
- You have to give an exact name of a bundle, using `Format: <GITREPO-NAME>-<BUNDLE_PATH> with all path separators replaced by "-" as` as mentioned in [this page](https://fleet.rancher.io/gitrepo-structure/#fleetyaml), which is not practical, because at the time of creating the fleet.yaml file, the user probably does not know what the GitRepo name is. An enhancement issue is available [here](https://github.com/rancher/fleet/pull/552#issuecomment-930645572) and it is under-work.
- External GitRepo dependency does not work correctly in that a Bundle will deploy even if one of its dependencies from another GitRepo is not installed. [An issue exists](https://github.com/rancher/fleet/issues/486) for that as well.

This means that you need probably to be aware of these problems and configure your fleet accordingly:
- Do not use external GitRepo dependencies
- Make sure you put the right names for your dependencies, knowing in advance what the gitrepo name will be

In order to reproduce the working and the invalid scenario, I assume the following:
- 2 downstream clusters
- 1 bundle `rancher-loggin` for Rancher BanzaiCloud Logging Operator that deploys to the other cluster
- my `contests` branch, which deploys all the necessary bundles into my target cluster `contests` (NOTE: Please make sure you do modify the cluster id in the GitRepo)
- my `contests-invalid` branch, that supposes the bundles for logging are coming from the `rancher-logging` bundle, and defines dependencies bases on `rancher-logging-rancher-logging` bundle (`rancher-logging` paht in the `rancher-logging` GitRepo)

For the `contests-invalid` branch, the idea is :
- first to check when `rancher-logging` only applies to the other cluster (it should have only one target cluster).
- Then, add the other cluster, using the following command: `k patch gitrepo rancher-logging -n fleet-default -p '{"spec":{"targets":[{"clusterName":"c-zl9zl"},{"clusterName":"c-lsj9p"}]}}' --type=merge` : observe how after a short while the dependency constraint will appear.
- You can switch back to one single target cluster: `k patch gitrepo rancher-logging -n fleet-default -p '{"spec":{"targets":[{"clusterName":"c-zl9zl"}]}}' --type=merge` 

## Observations
The `contests` branch will deploy successfully, but if someone looks closely, since the bundles in the GitRepo are ordered with `rancher-logging` before `rancher-logging-crd`, it tries for a short time to install it though the `rancher-logging-crd` is a non-present dependency.
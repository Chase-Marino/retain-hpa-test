### Overview

We are trying to replicate a migration scenario where src cluster hpa may be disabled/differ from target as we are scaling to 0.

## Expected results

We expect the src and target deployments replica count to be able to be controlled independently, by only their respective hpa.

## Actual results

Target cluster replica count is being continuously updated by karmada by the src deployment/hpas replica count.


## steps to reproduce

# create deployment, hpa
ka source apply -f deployment.yaml

ka source apply -f hpa.yaml

# note that this hpa has scaling disabled and minReplicas = maxReplicas. So all this hpa should be doing is changing the replica count to the minReplicas value (2 in this case)

# propagate to target cluster
ka source apply -f pp.yaml

# observe matching state in both clusters
ka source -n retain-test get deployment

ka member1 -n retain-test get deployment

# apply retain label to target cluster. replica count should still match in both clusters, should be able to observe label applied
ka source apply -f retain-label-op.yaml

# update target cluster hpa
ka source apply -f hpa.yaml

# note we are updating minReplicas/maxReplicas to 4, and hpa is still disabled. this should just scale the target deployment to 4

# observe deployment state
ka source -n retain-test get deployment -w

ka member1 -n retain-test get deployment -w

# we see that replica count is spamming between 2 and 4, when it should be constant at 4 in target cluster

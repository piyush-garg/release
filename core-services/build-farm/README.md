# Build Farm

This folder provides the assets on the [prow-cluster](https://api.ci.openshift.org:443) for CI [build-farm](../../clusters).

## pull secrets

If the pods generated by prowjobs need to use an image from the prow-cluster which may be protected by authentication, eg, `registry.svc.ci.openshift.org/ci/ci-operator:latest`, then we need to use [`imagePullSecrets`](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) in the pod or its service account.

* create `SA`, e.g., `ci/build01` on _prow-cluster_:

    > oc apply -f ./core-services/build-farm/admin_build01_rbac.yaml

* get the token for the `SA` 

    > oc sa get-token -n ci build01

* generate `.docker/config.json`:

    > docker login -u unused -p _token_ registry.svc.ci.openshift.org

* create `secret` on the _cluster in build farm_:

    > oc process -f ./clusters/build-clusters/01_cluster/openshift/ci-operator/regcred_secret_template.yaml -p build01_ci_reg_auth_value=_build01_ci_reg_auth_value_ | oc apply -f -

* [use the secret](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account) as `imagePullSecrets` in the SA to run the pod of a prowjob:

    > oc apply -f clusters/build-clusters/01_cluster/openshift/ci-operator/admin_ci-operator_rbac.yaml

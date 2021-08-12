# failk8s Package Respository

1. Add the packages you want to config.yaml (and other configuration)
2. Create the packages
   ```
   ./hack/build.sh
   ```
   NOTE: This will create a `develop` version of the repository.
3. When you're fine, release the repository, and version properly (use `git flow release <VERSION>`).
   Generate all the descriptors and packages in the OCI registry.
   ```
   ./hack/push.sh
   ```
4. Get ready for next version


## Deploy in cluster

1. Deploy kapp-controller 0.20.0+
   ```
   kubectl apply -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/latest/download/release.yml
   ```
2. Install this package repository
   ```
   kubectl apply -f target/k8s/repository.yaml
   ```

## Test the dev-platform

See up to date instructionns in [dev-platform](https://github.com/failk8s-packages/dev-platform) documentation. 

1. Create the dev-platform deployment files
   ```
   cat <<EOF | kubectl apply -f -
   ---
   apiVersion: v1
   kind: Namespace
   metadata:
   name: dev-platform
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
   name: dev-platform
   namespace: dev-platform
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
   name: dev-platform
   roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: cluster-admin
   subjects:
   - kind: ServiceAccount
      name: dev-platform
      namespace: dev-platform
   EOF 
   ```
2. Create the config for your cluster
   ```
   kubectl create secret generic dev-platform -n dev-platform --from-file=values.yaml=./profiles/override-jomorales.yaml -o yaml --dry-run=client | kubectl apply -f -
   ```
3. Install the package
   ```
   cat <<EOF | kubectl apply -n dev-platform -f -
   ---
   apiVersion: packaging.carvel.dev/v1alpha1
   kind: PackageInstall
   metadata:
   name: dev-platform
   spec:
   serviceAccountName: dev-platform
   packageRef:
      refName: dev-platform.dev.failk8s.com
      versionSelection:
         constraints: "0.0.0+develop"
         prereleases: {}
   values:
   - secretRef:
         name: dev-platform
   EOF
   ```

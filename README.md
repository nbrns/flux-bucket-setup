# Flux Bucket Setup (With S3)

The idea of this repository is to run [FluxCD](https://fluxcd.io/) without a git server but the [Bucket Source](https://fluxcd.io/docs/components/source/buckets/). This means, Flux bootstrapping takes place only using a Bucket Source. This allows local setups of Flux without a running git server.
We will use the Flux generic bucket type, however in this example it will be backed by an AWS S3. You can go for any S3 compatible storage, e.g. a local [minio](https://min.io/).

## Local Kubernetes Cluster Setup

Set up a local Kubernetes cluster e.g. using [minikube](https://minikube.sigs.k8s.io/docs/start/).

```bash
minikube start --driver=docker
```

## Flux Setup

Make sure to install [flux cli](https://fluxcd.io/docs/cmd/) first.

Instead of using the [flux bootstrap](https://fluxcd.io/docs/cmd/flux_bootstrap/) command, which will bootstrap a git repository by default, we will take the manual path.

Export the Flux [GOTK components](https://fluxcd.io/docs/components/)

```bash
mkdir -p clusters/local/flux-system
flux install --export > clusters/local/flux-system/gotk-components.yaml
```

Install the GOTK components on the cluster (install Flux CRDs and instantiate base controllers in namespace _flux-system_)

```bash
kubectl apply -f clusters/local/flux-system/gotk-components.yaml
```

## Setup The Bucket Source (S3 Example)

Create the bucket (make sure to install [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and configure it with `aws configure`)

```bash
aws s3api create-bucket --bucket flux-local-setup-test --region us-east-1
```
> **NOTE**: If you wish to use another bucket name, make sure to change the bucketName in the Bucket spec of [gotk-sync.yaml](clusters/local/flux-system/gotk-sync.yaml) as well.

Upload your manifests to the bucket
```bash
aws s3 cp --recursive ./clusters/ s3://flux-local-setup-test/clusters/
```

Edit and save the contents of the flux-credetials.yaml file. It should similar to this:
```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: flux-system-bucket-credentials
  namespace: flux-system
type: Opaque
data:
  # Paste your secret data here and think of using SOPS to encrypt secrets, cf. https://fluxcd.io/docs/guides/mozilla-sops/
  accesskey: <ACCESS_KEY>
  secretkey:  <ACCESS_KEY_SECRET>
```

Apply the secret
```bash
kubectl apply -f flux-credentials.yaml
```

Apply the source
```bash
kubectl apply -f clusters/local/flux-system/gotk-sync.yaml
```

Now Flux starts syncing and creates the demo application in [apps](./clusters/local/apps/) for you.

> **NOTE**: The source sync is usually established by the `flux bootstrap` command, which generates the necessary manifests. Thus, be careful when applying changes to the source sync, as it is essential for Flux to work.

## Cleanup

If desired, cleanup the local cluster by `minikube delete`.

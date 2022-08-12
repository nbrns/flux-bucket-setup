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
cd clusters/local/flux-system
flux install --export > gotk-components.yml
```

Install the GOTK components (install Flux CRDs and instantiate base controllers in namespace _flux-system_)

```bash
kubectl apply -f gotk-components.yml
```

## Setup The Bucket Source (S3)

Create the bucket

```bash

```

Create the secret

```yaml

```

Apply the secret

Create the secret

```bash

```

Apply the source

```bash

```

## Cleanup

If desired, cleanup the local cluster by `minikube delete`.

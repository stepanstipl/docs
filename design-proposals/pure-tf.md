# Pure Terraform to manage infrastructure

https://github.com/stepanstipl/gpii-pure

## What
Use only Terraform and it's providers to manage GCE/GKE infrastrucrure, with
simple Make wrapper.

## Motivation

The infrastructure automation has quite a few dependencies currently, both
required to be installed on the host (Ruby, Rake, docker-compose) and many in
Exekube's container (Terragrunt, Exekube itself). This makes running the
automation more difficult, but mainly introduces unnecessary complexity
and enforces opiniated patterns (like directory structure aka Terragrunt).
Also it's monolithical nature, with no well defined interfaces between
components, makes it difficult to replace or reuse parts of code. Some of these
dependecies (Exekube) are very young projects, with minimal history and no
community.

Simpler and well structured infrastructure code will allow us to move faster,
have better visibility and understanding of the solution, it is less prone
to errors and is more secure.

## Benefits
- Fewer dependencies
- Simplified project structure
- Everything packaged in Docker
- Consistent environment
- No local state
- Compatible with modern container based CIs (like Concourse or Google Cloud
  Build)
- More flexible to extend
- Easier to maintain

## Architecture notes

The structure of the solution is very simple, with well defined
interfaces (via Terrafrom variables) between components.

```
user/CI -> docker -> make -> terraform -> terraform-providers
```

In general it aims to follow Linux philosophy of *do one thing and do it well*.

Refer to the code repo for more details https://github.com/stepanstipl/gpii-pure.

### Helm Charts

Helm Charts will live in separate repository, have independent versioning and CI
workflow and be published to GCS based Helm repository.

### More complex code
This solution does not rule out inclusion of more complex code, let's say in
Ruby, when appropriate. But rather than being tangled into rest of the infra code,
it would live separately and be included via another docker container.

Good example might be performance tests. The tests would be packaged as anotther
docker container, with well-defined interface (parametes like URL, etc.), and
executed through make target.

## Use cases

### User
User does not need to do any setup apart from having Docker installed. Running
`docker run -it --rm -e TF_VAR_project=dev-xyz-1234 gpii-pure cluster-apply`
will bring up whole cluster up (alternative is 
`TF_VAR_project=dev-xyz-1234 make docker-cluster-apply` when one has the Makefile).

### Admins
Project management, apart from initial `admin-init`, should run under CI. For
development there's always possibility to start continaer with mounted directory
to be able to interactively work on code and benefit from consistent environment
at the same time.
Updating projects based on `admin/projects.tf` si matter one call -
`TF_VAR_project=root-123 make docker-admin-apply`.

### CI
Given everyhing is packaged in Docker container, no local state, and duality
of make steps (`cluster-apply` and `docker-cluster-apply`), it will be
very easy to run on any CI, but mainly on modern container based ones like
Concourse of Google Cloud Build.

## Out-of-scope

### State encryption & secrets
State is encryption is not explicitly addresed as part of this spike. State is 
encrypted using default GCS encryption, and should be easy to move to "Customer
Managed Key" using KMS.
Sound  strategy for secrets management would be to a) eliminate static secrets
from out infrastructure and b) don't expose secrets to automation code. Using
Terraform also offers partial support for sensitive data - some providers have
fileds that are marked as `sensitive` and don't appear in outputs, output
variables support same.

### User accounts and groups
While this code brings whole GCP Organizatuon under TF management, including IAM
permissions, it uses simple model of `administrators` group, that have
permissions across whole organization, and individual projects are then owned by
individual users. Actual structure and requirements to be discussed elswhere.

### Service accounts
The currnt code does not solve problem of passing service account credentials to
Terraform. Ideally it would run inside GCP and therefore allow us to use
service accounts for instances. Otherwise credentials can be passed as a file or
environemnt variable.

#$# Tests
Testing has not been addressed as part of this spike.

## FAQ

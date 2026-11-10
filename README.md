# Secure Container Build and ECR Pipeline

This project demonstrates a small containerized Node.js application and a CI
pipeline that builds the image, scans it with Trivy, and pushes it to Amazon
Elastic Container Registry (ECR). It is a DevOps portfolio project focused on
container builds, vulnerability scanning, and AWS image publishing.

## Project flow

1. Push a commit to `main`.
2. GitHub Actions checks out the source and installs Trivy on the runner.
3. Docker builds the application image, tagged with the commit SHA.
4. Trivy scans operating-system packages and application libraries for HIGH
   and CRITICAL vulnerabilities.
5. GitHub Actions authenticates to AWS and pushes the image to ECR with both
   the commit SHA and `latest` tags.

The workflow is in [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml).
It targets AWS region `us-east-1` and ECR repository `testing/security-scan`.
The scan currently uses `--exit-code 0`: it reports matching vulnerabilities
but does not block the image push. Change this to `--exit-code 1` when you want
the scan to act as a deployment gate.

## Local testing

### Build and run the application image

From the repository root, build the image using the current Alpine-based
Dockerfile and run it:

```bash
docker build -t app:v2 .
docker run --rm app:v2
```

The application prints a short verification message and exits.

### Compare the standard and Alpine Node.js images

The initial local comparison used the standard `node:22` image, followed by
the smaller `node:22-alpine` image now used in the Dockerfile. To reproduce
the comparison, temporarily change the Dockerfile's `FROM` line to
`node:22`, then build and scan:

```bash
docker build -t app:v1 .
trivy image --severity HIGH,CRITICAL app:v1
```

Then restore the `FROM` line to `node:22-alpine` and run:

```bash
docker build -t app:v2 .
trivy image --severity HIGH,CRITICAL app:v2
```

In the initial local test, the standard image reported a critical finding,
while the Alpine image reported no HIGH or CRITICAL findings. Vulnerability
results can change as image layers and Trivy's vulnerability database are
updated, so treat each scan's current output as authoritative. Alpine's
smaller footprint can reduce included packages, but it does not guarantee an
image is vulnerability-free.

To make a local scan fail when it finds HIGH or CRITICAL vulnerabilities, add
`--exit-code 1`:

```bash
trivy image --exit-code 1 --severity HIGH,CRITICAL app:v2
```

## GitHub Actions and AWS setup

For the workflow to push successfully:

- The ECR repository `testing/security-scan` must exist in `us-east-1`.
- The AWS credentials configured for the workflow must belong to the account
  that owns that ECR repository and have permission to authenticate and push.
- Add the credentials as GitHub Actions secrets named
  `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. Do not commit credentials
  to the repository.
- Push a commit to `main`, then review the run and its logs under the
  repository's **Actions** tab.

For a production setup, prefer GitHub Actions OIDC federation with an
appropriately scoped IAM role over long-lived AWS access keys.

## Files

- `app.js` - minimal Node.js application used to verify the container.
- `Dockerfile` - builds the application image from `node:22-alpine`.
- `.github/workflows/deploy.yml` - builds, scans, and pushes the image to ECR.

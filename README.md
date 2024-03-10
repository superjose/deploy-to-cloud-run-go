# deploy-to-cloud-run-go

An example project that shows you how to deploy to Google Cloud Run step by step using Pulumi, Go, and Google Cloud CLI.

1. This project was initialized with [GoWebly CLI](https://gowebly.org/).

- The GoWebly CLI was used too bootstrap a sample project.
- It is not required for this example.
- The only configuration folder lies inside the ./pulumi'

2. We modify the Dockerfile:
   a. We set golang to ver 1.22-alpine (Fixed in recent versions of Gowebly CLI)
   b. We add the `ENV HOME=/root` to the Dockerfile (Before the `ENTRYPOINT`). Otherwise the Docker image wouldn't run. [Here's why](https://stackoverflow.com/questions/71083833/docker-container-runs-locally-but-fails-on-cloud-run)

3.

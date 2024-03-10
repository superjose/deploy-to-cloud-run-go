# deploy-to-cloud-run-go

An example project that shows you how to deploy to Google Cloud Run step by step using Pulumi, Go, and Google Cloud CLI.

1. This project was initialized with [GoWebly CLI](https://gowebly.org/).

- The GoWebly CLI was used too bootstrap a sample project.
- It is not required for this example.
- The only configuration folder lies inside the ./pulumi'

2. We modify the Dockerfile:
   a. We set golang to ver 1)22-alpine (Fixed in recent versions of Gowebly CLI)
   b. We add the `ENV HOME=/root` to the Dockerfile (Before the `ENTRYPOINT`). Otherwise the Docker image wouldn't run. [Here's why](https://stackoverflow.com/questions/71083833/docker-container-runs-locally-but-fails-on-cloud-run)

## Get the CLIs:

3. [Create a Google Cloud Account](https://cloud.google.com/?hl=en).

4. [Install Google Cloud CLI](https://cloud.google.com/sdk/docs/install)

5. [Install Pulumi CLI](https://www.pulumi.com/docs/install/)

6. [Install Docker](https://www.docker.com/products/docker-desktop/) - We need it to build the Docker container.

## Bootstrap the project:

![Project Overview](https://dev-to-uploads.s3)amazonaws.com/uploads/articles/ancnkjl3ml5plgem9hy9)png)

7. Create a `pulumi` directory.

8. Run `pulumi new go` to initialize a pulumi project with Go (It can be your language of choice).

9. Navigate to the pulumi directory.

10. Navigate to [cloud.google.com](cloud.google.com) and create a new project.

![Project Id within Google Cloud Platform.](https://dev-to-uploads.s3)amazonaws.com/uploads/articles/xjila1mekxiiqrjr1t1k.png)

Take note of the `project-id` (Usually the project's name).

## Generate the necessary permissions using gcloud CLI

11. Login with the Google Auth CLI:

```sh
gcloud auth login
```

Open the link that will show up and finish logging in.

12. (Optional) Set the project in google cloud CLI (Can be changed anytime). This saves you from passing `--project [PROJECT-ID]` into every `gcloud` command.

If your machine has multiple GCP projects, skip this step and pass the `--project` flag into every `gcloud` command.

13. Create a service account (The account that Pulumi will connect to):

```sh
gcloud iam service-accounts create pulumi-gcp --description="Pulumi GCP"
```

14. Download the credentials for the service accounts and store them locally (Remember to replace `[PROJECT-ID]` with your GCP Project Id):

```sh
gcloud iam service-accounts keys create ~/keys/gcp/pulumi-service-account-key-file.json --iam-account=pulumi-gcp@[PROJECT-ID].iam.gserviceaccount.com
```

15. Set Pulumi's gcp credentials config path:
    (This will connect the service account with Pulumi)

```
pulumi config set gcp:credentials ~/keys/gcp/pulumi-service-account-key-file.json
```

16. Set the GCP Project by doing:

```sh
pulumi config set gcp:project [PROJECT-ID]
```

17. Create a `roles.gcp.yml` file (Inside the `pulumi` dir) and add the required permissions in `includedPermissions`:

![roles.gcp.yml location](https://dev-to-uploads.s3)amazonaws.com/uploads/articles/p4gj181hl6kjy06sy2ec.png)

18. Create the `pulumi_admin_role` with the file above:
    (We assume we're running this code from the `pulumi` directory)

```sh
gcloud iam roles create pulumi_admin_role --project=[PROJECT-ID] --file='./roles.gcp.yml'
```

19. In case you need to make edits, change the file and use:

```sh
gcloud iam roles update pulumi_admin_role --project=[PROJECT-ID] --file='./roles.gcp.yml'
```

20. We're also adding the `serviceAccountAdmin` role (I haven't found a better way) (Otherwise we'd get 403 errors when refreshing and updating in Pulumi)<sup>1</sup>

```sh
gcloud projects add-iam-policy-binding [PROJECT-ID] --role roles/iam.serviceAccountAdmin   --member serviceAccount:pulumi-gcp@[PROJECT-ID].iam.gserviceaccount.com
```

21. Our `main.go` in the `pulumi` directory:
    (Check the code for comments!)

21.1 We enable the required services (Artifact Registry, and Cloud Run).

21.2 Artifact Registry is used to host the docker container image.

21.3 Cloud Runner will launch the Docker image from Artifact Registry.

21.4 We build the docker image locally (We specify the platform in case you're using an ARM chip like M1, M2, Snapdragon SQ, X Elite, etc.)

21.5 We create a chain of "DependsOn" to notify Pulumi:
21.5.1 Services need to be enabled first
21.5.1 We create the Artifact Repository
21.5.1 We build the docker image and push it to Artifact Registry.
21.5.1 We pull the Docker Image from Artifact Registry and run it.
21.5.1 We add IAM permissions so it can be accessed from anywhere.

22. Update the `ENV` in your Dockerfile:
    There's a [known issue](https://stackoverflow.com/a/73926956/1057052), and [here](https://cloud.google.com/run/docs/issues#home) in which you need to export your `HOME` environment variable to `/root`;

```Dockerfile
# Set the ENV HOME before your ENTRYPOINT.
ENV HOME=/root

# This is specific to your project.
ENTRYPOINT ["/whatever-is-your-entrypoint"]
```

23. Create a .env file in the pulumi directory:
    Set the **\*full path** to the one you saved on `11)`

```sh
GOOGLE_CREDENTIALS_FILE_PATH="/Users/myusername/keys/gcp/pulumi-service-account-key-file.json"
```

24. Run `pulumi up`

And you should be up and going!

25. The `go.mod`
    If you include this `go.mod` Run `go tidy`, and this will fetch all the packages for you

## Footnotes

<sup>1</sup>I fought against permissions for 5 days. The `serviceAccountAdmin` predefined GCP role brought in the additional permissions needed.

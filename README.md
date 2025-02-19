
# Instant deployments for Google Cloud Run

Deploy to Cloud Run without building your sources into a container.

The idea is to load your app straight from Cloud Storage, instead of packaging it in the deployed container image. The Cloud Run service is configured use one of Cloud Run's runtime base images and it mounts the Cloud Storage bucket in the location where it expects the app to be.

## Node.js quickstart

The following instructions are for a Node.js app. For other languages, see the section at the bottom of this README. 

### Install and configure Google Cloud CLI

See instructions to [install the `gcloud` CLI](https://cloud.google.com/sdk/docs/install), then:

```
gcloud init
```

### Create a Cloud Storage bucket to store your app:

```
gcloud storage buckets create gs://BUCKET_NAME
```

Replace `BUCKET_NAME` with a unique name, for example `steren-app-123`.

### Copy your app's files in the bucket

In this example, a Node.js app is in the `app` folder.

If the app needs any dependencies, make sure to run `npm install` first (on a x86 Linux system).

Run the following to copy it into the bucket:

```
gcloud storage cp -r app/* gs://BUCKET_NAME/
```

### Deploy the Cloud Run service

Open `service.yaml` and replace `BUCKET_NAME` with your bucket name.

Run the following to deploy the service:

```
gcloud run services replace service.yaml
```

Make the service public: 

```
  gcloud run services add-iam-policy-binding instant-app --region us-central1 \
    --member="allUsers" \
    --role="roles/run.invoker"
```

### Enjoy!

Your Node.js app is now serving at the URL of the Cloud Run service

### Updating your code and the app

Re-run the following to update the code in Cloud Storage:

```
gcloud storage cp -r app/* gs://BUCKET_NAME/
```

The code will be picekd on any new instance of your Cloud Run service.

If you want to force existing instances of your service to pick up the new code, you can simply create a new revision by updating a environment variable:

```
gcloud run services update instant-app --region us-central1 --update-env-vars DATE=$(date +%Y-%m-%d_%H:%M:%S)
```


## Using other languages

Change the following lines in `service.yaml`:

```
      - image: us-central1-docker.pkg.dev/serverless-runtimes/google-22/runtimes/nodejs22
        command:
        - node
        - index.js
```

Replace the image with [one of Cloud Run's base images](https://cloud.google.com/run/docs/configuring/services/runtime-base-images#how_to_obtain_base_images), or even your own.

Replace the command with the start command for your language.

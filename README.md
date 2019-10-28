<div>
  <h1 align="center">
      :custard: Flan Scan :custard:
  </h1>
</div>

Flan Scan is a lightweight network vulnerability scanner. With Flan Scan you can easily find open ports on your network, identify services and their version, and get a list of relevant CVEs affecting your network. 


Set Up
------
1. Make sure you have docker setup:
```bash
$ docker --version
```

2. Add the list of IP addresses or CIDRS you wish to scan to `shared/ips.txt`.

3. Build the container (if not downloading from a registry):
```bash
$ make build
```

4. Start scanning!
```bash
$ make start
```

When the scan finishes you will find a Latex report of the summarizing the scan in `shared/results`. You can also see the raw XML output from Nmap in `shared/xml_files`.


Pushing Results to the Cloud
----------------------------

We currently support pushing Latex reports and raw XML Nmap output to a GCS Bucket or to an AWS S3 Bucket. Flan Scan requires 2 environment variables to push results to the cloud. The first is `upload` which takes one of two values `gcp` or `aws`. The second is `bucket` which is the name of the S3 or GCS Bucket you with to upload the results to. To set the environment variables, run the container setting the environment variables after running `make build`.
```bash
$ docker run --name <container-name> \
             -v $(pwd)/shared:/shared \
             -e upload=<gcp or aws> \
             -e bucket=<bucket-name> \
             flan_scan
```

Below are some exmples for adding the necessary AWS or GCP authentication keys as environmenta variables in container. However, this can also be accomplished with a secret in Kubernetes that exposes the necessary environment variables or with other secrets management tools.


### Example GCS Bucket Configuration

Copy your GCS private key for a service account to the `/shared` file
```bash
$ cp <path-to-local-gcs-key>/key.json /shared/
```

Run the container setting the `GOOGLE_APPLICATION_CREDENTIALS` environment variable as the path to the GCS Key

```bash
$ docker run --name <container-name> \
             -v $(pwd)/shared:/shared \
             -e upload=gcp \
             -e bucket=<bucket-name> \
             -e GOOGLE_APPLICATION_CREDENTIALS=/shared/key.json
             flan_scan 
```

### Example AWS S3 Bucket Configuration

Set the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables to the corresponding variables for your S3 service account.

```bash
docker run --name <container-name> \
           -v $(pwd)/shared:/shared \
           -e upload=aws \
           -e bucket=<s3-bucket-name> \
           -e AWS_ACCESS_KEY_ID=<your-aws-access-key-id> \
           -e AWS_SECRET_ACCESS_KEY=<your-aws-secret-access-key> \
           flan_scan


```

Deploying on Kubernetes
-----------------------

When deploying Flan Scan to a container orchestration system, such as Kubernetes, you must ensure that the container has access to a file called `ips.txt` at the directory `/`. In Kubernetes this can be done with a ConfigMap which will mount a file on your local filesystem as a volume that the container can access once deployed. The `kustomization.yaml` has an example of how to create a ConfigMap called `shared-files` that is mounted as a volume in the `deployment.yaml` file also in this repo. 

Here are some easy steps to deploy Flan Scan on Kubernetes:
1. To create the the ConfigMap add a path to a local `ips.txt` file in `kustomization.yaml` and then run `kubectl apply -k .`. 
2. Now run `kubectl get configmap` to make sure the ConfigMap was created properly. 
3. Set the necessary environment variables and secrets for your cloud provider as described in the previous section within `deployment.yaml`.
4. Now run `kubectl apply -f deployment.yaml`

Flan Scan should run successfully on Kubernetes! 

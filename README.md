# Guide to SeaweedFS and Minio CLI on GKE

![image.png](image.png)

Image ref: [Seaweedfs Distributed Storage Part 1: Introduction. | by Ali Hussein Safar | Medium](https://medium.com/@ahsifer/seaweedfs-25640728775c)

## Deploying SeaweedFS on GKE via Helm Chart

Noted: Assuming you’re connected to the kubernetes cluster

1. **Git Clone** 

```hcl
git clone https://github.com/GGital/SeaweedFS-GKE-manual.git
```

1. **Setup Helm repository**

```hcl
helm repo add seaweedfs https://seaweedfs.github.io/seaweedfs/helm
helm repo update
```

1. **Deploying to cluster**

```hcl
kubectl create namespace seaweedfs
kubectl apply -f lb-svc
helm install my-seaweed seaweedfs/seaweedfs -n seaweedfs -f helm-value.yaml
```

### References for more helm values

- https://artifacthub.io/packages/helm/seaweedfs/seaweedfs
- https://github.com/seaweedfs/seaweedfs/tree/master/k8s/charts/seaweedfs

## How to access services

1. **Get external-IP**

```hcl
kubectl get svc seaweedfs-filer-public seaweedfs-s3-public
```

- Services are accessible at load balancer’s external IP with the specified ports
    - filer service: http://< filer-lb-external-ip >:8888
    - s3 api service: http://< s3-lb-external-ip >:8333

## Connecting minio CLI to Seaweedfs S3 API

1. Running instance of minio CLI with docker

```hcl
docker run --rm -it --entrypoint=/bin/sh minio/mc

# You may mount the volume with -v to see downloaded file on local

# Example docker run --rm -it --entrypoint=/bin/sh -v /download:/data minio/mc
```

Now, the terminal will accept mc alias commands 

1. Setting alias to s3 api service

```hcl
mc alias set seaweed http://<s3-lb-external-ip>:8333 <ACCESS_KEY> <SECRET_KEY>

# Default ACCESS_KEY is admin, SECRET_KEY is adminpassword
```

## Testing integration of minio CLI and seaweedfs

 

| **Step** | **Command** | **What it tests** |
| --- | --- | --- |
| **1. Create Bucket** | `mc mb seaweed/test-bucket` | Write permission to the Filer metadata. |
| **2. Upload File** | `echo "hello seaweed" > test.txt` 
 `mc cp test.txt seaweed/test-bucket/` | Data transfer to Volume servers. |
| **3. List Files** | `mc ls seaweed/test-bucket/` | Metadata retrieval from the Filer. |
| **4. Download File** | `mc cp seaweed/test-bucket/test.txt ./test-download.txt` | Read path from Volume servers. |
| **5. Cleanup** | `mc rm --recursive --force seaweed/test-bucket` | Delete/Garbage collection logic. |

## Notion

Notion Link : https://mewing-loaf-905.notion.site/Guide-to-SeaweedFS-and-Minio-CLI-on-GKE-2e00f518934580ba921cff0ad19095da?source=copy_link
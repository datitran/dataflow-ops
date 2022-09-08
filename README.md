# Template for Prefect deployments with Continuous Deployment GitHub Actions workflow and one-click agent deployment

The goal of this recipe is to have one Prefect agent (running on AWS ECS Fargate) with shared core package dependencies per project. This means that:

- the pip packages such as pandas, numpy, scikit learn etc. are baked into an ECR image and this image gets used to deploy an agent process
- the flow deployments can have custom module dependencies which are packaged alongside flow code to S3
- the flow deployments are created automatically from CI/CD but they can also be created from your local machine and this will work the same way


Make sure to adjust your AWS account ID and your default region in the ``task-definition.json`` file.


# Deployment CLI examples based on your platform, storage and infrastructure

<details>
  <summary>Table with examples</summary>
  

| Storage Block | Infrastructure Block | End Result | CLI Build Command for hello.py flow with flow function hello | Platform |
| --- | --- | --- | --- | --- |
| N/A | N/A | Local storage and local process on the same machine from which you created a deployment | prefect deployment build hello.py:hello -n implicit -q dev | Local/VM |
| N/A | N/A | Local storage and local process on the same machine from which you created a deployment — but with version and storing the output YAML manifest with the given file name in the deploy directory  | prefect deployment build hello.py:hello -n implicit-with-version -q dev -v github_sha -o deploy/implicit_with_version.yaml | Local/VM |
| N/A | -ib process/dev | Local storage and local process on the same machine from which you created a deployment, but in contrast to the example from the first row, this requires you to create this Process block with name dev beforehand explicitly, rather than implicitly letting Prefect create it for you as anonymous block | prefect deployment build hello.py:hello -n implicit -q dev -ib process/dev | Local/VM |
| N/A | -ib process/dev | Local storage and local process block but overriding the default environment variable to set log level to debug via --override flag | prefect deployment build hello.py:hello -n implicit -q dev -ib process/dev --override env.PREFECT_LOGGING_LEVEL=DEBUG | Local/VM |
| N/A | --infra process | Local storage and local process on the same machine from which you created a deployment, but in contrast to the example in the first row, it explicitly specifies that you want to use process block; the result is exactly the same, i.e. Prefect will create an anonymous Process block | prefect deployment build hello.py:hello -n implicit -q dev --infra process | Local/VM |
| -sb s3/dev | -ib process/dev | S3 storage block and local Process block - this setup allows you to use a remote agent e.g. running on an EC2 instance; any flow run from this deployment will run as a local process on that VM and Prefect will pull code from S3 at runtime | prefect deployment build hello.py:hello -n s3-process -q dev -sb s3/dev -ib process/dev | AWS S3 + EC2 |
| -sb s3/dev | -ib docker-container/dev | S3 storage block and DockerContainer block - this setup allows you to use a remote agent e.g. running on an EC2 instance; any flow run from this deployment will run as a docker container on that VM and Prefect will pull code from S3 at runtime | prefect deployment build hello.py:hello -n s3-docker -q dev -sb s3/dev -ib docker-container/dev | AWS S3 + EC2 |
| -sb s3/dev | -ib kubernetes-job/dev | S3 storage block and KubernetesJob block - this setup allows you to use a remote agent running as Kubernetes deployment e.g. running on an AWS EKS cluster; any flow run from this deployment will run as a Kubernetes job pod within that cluster and Prefect will pull code from S3 at runtime | prefect deployment build hello.py:hello -n s3-k8s -q dev -sb s3/dev -ib kubernetes-job/dev | AWS S3 + EKS |
| -sb gcs/dev | -ib process/dev | GCS storage block and local Process block - this setup allows you to use a remote agent e.g. running on Google Compute Engine instance; any flow run from this deployment will run as a local process on that VM and Prefect will pull code from GCS at runtime | prefect deployment build hello.py:hello -n gcs-process -q dev -sb gcs/dev -ib process/dev | GCP GCS + GCE |
| -sb gcs/dev | -ib docker-container/dev | GCS storage block and DockerContainer block - this setup allows you to use a remote agent e.g. running on Google Compute Engine instance; any flow run from this deployment will run as a docker container on that VM and Prefect will pull code from GCS at runtime | prefect deployment build hello.py:hello -n gcs-docker -q dev -sb gcs/dev -ib docker-container/dev | GCP GCS + GCE |
| -sb gcs/dev | -ib kubernetes-job/dev | GCS storage block and KubernetesJob block - this setup allows you to use a remote agent running as Kubernetes deployment e.g. running on GCP GKE cluster; any flow run from this deployment will run as a Kubernetes job pod within that cluster and Prefect will pull code from GCS at runtime | prefect deployment build hello.py:hello -n gcs-k8s -q dev -sb gcs/dev -ib kubernetes-job/dev | GCP GCS + GKE |
| -sb azure/dev | -ib process/dev | Azure storage block and local Process block - this setup allows you to use a remote agent e.g. running on Azure VM instance; any flow run from this deployment will run as a local process on that VM and Prefect will pull code from Azure storage at runtime | prefect deployment build hello.py:hello -n az-process -q dev -sb azure/dev -ib process/dev | Azure Blob Storage + Azure VM |
| -sb azure/dev | -ib docker-container/dev | Azure storage block and DockerContainer block - this setup allows you to use a remote agent e.g. running on Azure VM instance; any flow run from this deployment will run as a docker container on that VM and Prefect will pull code from Azure storage at runtime | prefect deployment build hello.py:hello -n az-docker -q dev -sb azure/dev -ib docker-container/dev | Azure Blob Storage + Azure VM |
| -sb azure/dev | -ib kubernetes-job/dev | GCS storage block and KubernetesJob block - this setup allows you to use a remote agent running as Kubernetes deployment e.g. running on Azure AKS cluster; any flow run from this deployment will run as a Kubernetes job pod within that cluster and Prefect will pull code from Azure storage at runtime | prefect deployment build hello.py:hello -n az-k8s -q dev -sb azure/dev -ib kubernetes-job/dev | Azure Blob Storage + AKS |

</details> 

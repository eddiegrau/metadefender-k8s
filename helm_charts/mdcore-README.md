
Metadefender core
===========

This is a Helm chart for deploying MetaDefender Core (https://www.opswat.com/products/metadefender/core) in a Kubernetes cluster

This chart can deploy the following depending on the provided values:
- One or mode MD Core instances 
- A PostgreSQL database instance pre-configured to be used by MD Core

In addition to the chart, we also provide a number of values files for specific scenarios:
- mdcore-aws-eks-values.yml - for deploying in an AWS environment using Amazon EKS
- mdcore-azure-aks-values.yml - for deploying in an Azure environment using AKS

## Installation

### From source
MD Core can be installed directly from the source code, here's an example using the generic values:
```console
git clone https://github.com/OPSWAT/metadefender-k8s.git metadefender
cd metadefender/helm_carts
helm install my_mdcore ./mdcore
```

### From the GitHub helm repo
The installation can also be done using the helm repo which is updated on each release:
 ```console
helm repo add mdk8s https://opswat.github.io/metadefender-k8s/
helm repo update mdk8s
helm install my_mdcore mdk8s/metadefender_core
```

## Operational Notes
The entire deployment can be customized by overwriting the chart's default configuration values. Here are a few point to look out for when changing these values:
- Sensitive values (like credentials and keys) are saved in the Kubernetes cluster as secrets and are not deleted when the chart is removed and they can be reused for future deployments
- Credentials that are not explicitly set (passwords and the api key) and do not already exist as k8s secrets will be randomly generated, if they are set, the respective k8s secret will be updated or created if it doesn't exist
- **The license key value is mandatory**, if it's left unset or if it's invalid, the MD Core instance will report as "unhealthy" and it will be restarted
- The configured license should have a sufficient number of activations for all pod running MD Core, each pod counts as 1 activation. Terminating pods will also deactivate the respective MD Core instance.
- By default, a PostgreSQL database is deployed alongside the MD Core deployment with the same credentials as set in the values file
- In a production environment it's recommended to use an external service for the database (like Amazon RDS) and set `deploy_with_core_db` to false in order to not deploy an in-cluster database
- The deployed MD Core pod has a startup container that will wait for a database connection before allowing MD Core to start

## Configuration

The following table lists the configurable parameters of the Metadefender core chart and their default values.

| Parameter                | Description             | Default        |
| ------------------------ | ----------------------- | -------------- |
| `mdcore_user` | Initial admin user for the MD Core web interface | `"admin"` |
| `mdcore_password` | Initial admin password for the MD Core web interface, if not set it will be randomly generated | `null` |
| `core_db_user` | PostgreSQL database username | `"postgres"` |
| `core_db_password` | PostgreSQL database password, if not set it will be randomly generated | `null` |
| `mdcore_api_key` | 36 character API key used for the MD Core REST API, if not set it will be randomly generated | `null` |
| `mdcore_license_key` | A valid license key, **this value is mandatory** | `"<SET_LICENSE_KEY_HERE>"` |
| `activation_server` | URL to the OPSWAT activation server, this value should not be changed | `"activation.dl.opswat.com"` |
| `MDCORE_REST_PORT` | Default port for the MD Core service | `"8008"` |
| `MDCORE_DB_MODE` | Database mode | `"4"` |
| `MDCORE_DB_TYPE` | Database type | `"remote"` |
| `MDCORE_DB_HOST` | Hostname / entrypoint of the database, this value should be changed any if using an external database service | `"postgres-core"` |
| `MDCORE_DB_PORT` | Port for the PostgreSQL Database | `"5432"` |
| `deploy_with_core_db` | Enable or disable the local in-cluster PostgreSQL database | `true` |
| `persistance_enabled` |  | `true` |
| `storage_provisioner` |  | `"hostPath"` |
| `storage_name` |  | `"hostPath"` |
| `storage_node` |  | `"minikube"` |
| `hostPathPrefix` | If `deploy_with_core_db` is set to true, this is the absolute path on the node where to keep the database filesystem for persistance | `"mdcore-storage"` |
| `environment` | Deployment environment type, the default `generic` value will not configure or provision any additional resources in the cloud provider (like load balancers), other values: `aws_eks_fargate` | `"generic"` |
| `install_alb` | If set to true and `environment` is set to `aws_eks_fargate`, an ALB ingress controller will be installed | `true` |
| `eks_cluster_name` | Name of the EKS cluster, mandatory only if `environment` is set to `aws_eks_fargate` | `null` |
| `app_name` | Application name, it also sets the namespace on all created resources and replaces `<APP_NAME>` in the ingress host (if the ingress is enabled) | `"default"` |
| `core_ingress.host` | Hostname for the publicly accessible ingress, the `<APP_NAME>` string will be replaced with the `app_name` value | `"<APP_NAME>-mdss.local"` |
| `core_ingress.service` | Service name where the ingress should route to, this should be left unchanged | `"md-core"` |
| `core_ingress.port` | Port where the ingress should route to | `8008` |
| `core_ingress.enabled` | Enable or disable the ingress creation | `false` |
| `core_ingress.class` | Sets the ingress class | `"nginx"` |
| `core_docker_repo` |  | `"opswat"` |
| `core_components.postgres-core.name` |  | `"postgres-core"` |
| `core_components.postgres-core.image` |  | `"postgres"` |
| `core_components.postgres-core.env` |  | `[{"name": "POSTGRES_PASSWORD", "valueFrom": {"secretKeyRef": {"name": "mdcore-postgres-cred", "key": "password"}}}, {"name": "POSTGRES_USER", "valueFrom": {"secretKeyRef": {"name": "mdcore-postgres-cred", "key": "user"}}}]` |
| `core_components.postgres-core.ports` |  | `[{"port": 5432}]` |
| `core_components.postgres-core.is_db` |  | `true` |
| `core_components.postgres-core.persistentDir` |  | `"/var/lib/postgresql/data"` |
| `core_components.md-core.name` |  | `"md-core"` |
| `core_components.md-core.image` | Overrides the default docker image for the MD Core service, this value can be changed if you want to set a different version of MD Core | `"opswat/metadefendercore-debian:5.0.1"` |
| `core_components.md-core.replicas` | Sets the number of replicas if you want to have multiple MD Core instances | `1` |
| `core_components.md-core.env` |  | `[{"name": "MD_USER", "valueFrom": {"secretKeyRef": {"name": "mdcore-cred", "key": "user"}}}, {"name": "MD_PWD", "valueFrom": {"secretKeyRef": {"name": "mdcore-cred", "key": "password"}}}, {"name": "MD_INSTANCE_NAME", "valueFrom": {"fieldRef": {"fieldPath": "metadata.name"}}}, {"name": "APIKEY", "valueFrom": {"secretKeyRef": {"name": "mdcore-api-key", "key": "value"}}}, {"name": "LICENSE_KEY", "valueFrom": {"secretKeyRef": {"name": "mdcore-license-key", "key": "value"}}}, {"name": "DB_USER", "valueFrom": {"secretKeyRef": {"name": "mdcore-postgres-cred", "key": "user"}}}, {"name": "DB_PWD", "valueFrom": {"secretKeyRef": {"name": "mdcore-postgres-cred", "key": "password"}}}]` |
| `core_components.md-core.ports` |  | `[{"port": 8008}]` |
| `core_components.md-core.service_type` | Sets the service type for MD Core service (ClusterIP, NodePort, LoadBalancer) | `"ClusterIP"` |
| `core_components.md-core.extra_labels.aws-type` | If `aws-type` is set to `fargate`, the MD Core pod will be scheduled on an AWS Fargate virtual node (if a fargate profile is provisioned and configured) | `"fargate"` |
| `core_components.md-core.resources.requests.memory` | Minimum reserved memory | `"4Gi"` |
| `core_components.md-core.resources.requests.cpu` | Minimum reserved cpu | `"1.0"` |
| `core_components.md-core.resources.limits.memory` | Maximum memory limit | `"8Gi"` |
| `core_components.md-core.resources.limits.cpu` | Maximum cpu limit | `"1.0"` |
| `core_components.md-core.livenessProbe.httpGet.path` | Health check endpoint | `"/readyz"` |
| `core_components.md-core.livenessProbe.httpGet.port` | Health check port | `8008` |
| `core_components.md-core.livenessProbe.initialDelaySeconds` |  | `10` |
| `core_components.md-core.livenessProbe.periodSeconds` |  | `3` |
| `core_components.md-core.livenessProbe.timeoutSeconds` |  | `5` |
| `core_components.md-core.livenessProbe.failureThreshold` |  | `3` |
| `core_components.md-core.strategy.type` |  | `"RollingUpdate"` |
| `core_components.md-core.strategy.rollingUpdate.maxSurge` |  | `0` |
| `core_components.md-core.sidecars` | Configuration for the activation-manager sidecar | `[{"name": "activation-manager", "image": "alpine", "envFrom": [{"configMapRef": {"name": "mdcore-env"}}], "env": [{"name": "APIKEY", "valueFrom": {"secretKeyRef": {"name": "mdcore-api-key", "key": "value"}}}, {"name": "LICENSE_KEY", "valueFrom": {"secretKeyRef": {"name": "mdcore-license-key", "key": "value"}}}], "command": ["/bin/sh", "-c"], "args": ["apk add curl jq\nstop() {\n  echo 'Deactivating using the MD Core API'\n  curl -H \"apikey: $APIKEY\" -X POST \"https://localhost:$REST_PORT/admin/license/deactivation\"\n  echo 'Deactivating using activation server API'\n  curl -X GET \"https://$ACTIVATION_SERVER/deactivation?key=$LICENSE_KEY&deployment=$DEPLOYMENT\"\n  exit 0\n}\ntrap stop SIGTERM SIGINT SIGQUIT\n\nuntil [ -n $DEPLOYMENT ] && [ $DEPLOYMENT != null ]; do\n    echo 'Checking...'\n    export DEPLOYMENT=$(curl --silent -H \"apikey: $APIKEY\" \"http://localhost:$REST_PORT/admin/license\" | jq -r \".deployment\")\n    echo \"Deployment ID: $DEPLOYMENT\"\n    sleep 1\ndone\necho \"Waiting for termination signal...\"\nwhile true; do sleep 1; done\necho \"MD Core pod finished, exiting\"\nexit 0\n"]}]` |
| `core_components.md-core.initContainers` |  | `[{"name": "check-db-ready", "image": "postgres", "envFrom": [{"configMapRef": {"name": "mdcore-env"}}], "command": ["sh", "-c", "until pg_isready -h $DB_HOST -p $DB_PORT; do echo waiting for database; sleep 2; done;"]}]` |
| `podAnnotations` |  | `{}` |
| `podSecurityContext` |  | `{}` |
| `securityContext` |  | `{}` |
| `nodeSelector` |  | `{}` |
| `tolerations` |  | `[]` |
| `affinity` |  | `{}` |



---
_Documentation generated by [Frigate](https://frigate.readthedocs.io)._


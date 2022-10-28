
Metadefender ICAP Server
===========

This is a Helm chart for deploying MetaDefender ICAP (https://www.opswat.com/products/metadefender/icap) in a Kubernetes cluster

This chart can deploy the following depending on the provided values:
- One or mode MD ICAP Server instances 

## Installation

### From source
MD ICAP Server can be installed directly from the source code, here's an example using the generic values:
```console
git clone https://github.com/OPSWAT/metadefender-k8s.git metadefender
cd metadefender/helm_charts
helm install my-mdicap ./icap
```

### From the GitHub helm repo
The installation can also be done using the helm repo which is updated on each release:
```console
helm repo add mdk8s https://opswat.github.io/metadefender-k8s/
helm repo update mdk8s
helm install my_mdicapsrv mdk8s/metadefender_icap_server
```

## Operational Notes
The entire deployment can be customized by overwriting the chart's default configuration values. Here are a few point to look out for when changing these values:
- Sensitive values (like credentials and keys) are saved in the Kubernetes cluster as secrets and are not deleted when the chart is removed and they can be reused for future deployments
- Credentials that are not explicitly set (passwords and the api key) and do not already exist as k8s secrets will be randomly generated, if they are set, the respective k8s secret will be updated or created if it doesn't exist
- **The license key value is mandatory**, if it's left unset or if it's invalid, the MD ICAP Server instance will report as "unhealthy" and it will be restarted
- The configured license should have a sufficient number of activations for all pod running MD ICAP Server, each pod counts as 1 activation. Terminating pods will also deactivate the respective MD ICAP Server instance.

## Configuration

The following table lists the configurable parameters of the Metadefender ICAP chart and their default values.

| Parameter                | Description             | Default        |
| ------------------------ | ----------------------- | -------------- |
| `mdicapsrv_user` | Initial admin user for the MD ICAP Server web interface | `"admin"` |
| `mdicapsrv_password` | Initial admin password for the MD ICAP Server web interface, if not set it will be randomly generated | `null` |
| `mdicapsrv_api_key` | 36 character API key used for the MD ICAP Server REST API, if not set it will be randomly generated | `null` |
| `mdicapsrv_license_key` | A valid license key, **this value is mandatory** | `"<SET_LICENSE_KEY_HERE>"` |
| `activation_server` | URL to the OPSWAT activation server, this value should not be changed | `"activation.dl.opswat.com"` |
| `MDICAPSRV_REST_PORT` | Default REST port for the MD ICAP Server service | `"8048"` |
| `MDICAPSRV_ICAP_PORT` | Default ICAP port for the MD ICAP Server service | `"1344"` |
| `MDICAPSRV_ICAPS_PORT` | Default ICAPS port for the MD ICAP Server service | `"11344"` |
| `MDICAPSRV_CERT_PATH` | MD ICAP Server container will mount certificate to the path | `"/cert"` |
| `MDICAPSRV_IMPORT_CONF_FILE` | Initial configuration user management, global settings, server profiles, security rules, security config (https, icaps), certificates config, history config, audit log config and session timeout configuration | `"/opt/opswat/mdicapsrv-config.json"` |
| `persistance_enabled` | Set to false to not create any volumes or host paths in the deployment, all storage will be ephemeral | `true` |
| `storage_provisioner` | Available storage providers | `"hostPath"` |
| `storage_name` | Available storage providers | `"hostPath"` |
| `storage_node` | Available storage providers | `"minikube"` |
| `hostPathPrefix` | This is the absolute path on the node where to keep the data filesystem for persistance | `"mdicapsrv-storage"` |
| `environment` | Deployment environment type, the default `generic` value will not configure or provision any additional resources in the cloud provider (like load balancers), other values: `aws_eks_fargate` | `"generic"` |
| `app_name` | Application name, it also sets the namespace on all created resources and replaces `<APP_NAME>` in the ingress host (if the ingress is enabled) | `"default"` |
| `icap_ingress.host` | Hostname for the publicly accessible ingress, the `<APP_NAME>` string will be replaced with the `app_name` value | `"<APP_NAME>-mdicapsrv.k8s"` |
| `icap_ingress.service` | Service name where the ingress should route to, this should be left unchanged | `"md-icapsrv"` |
| `icap_ingress.rest_port` | Port where the ingress should route to | `8048` |
| `icap_ingress.enabled` | Enable or disable the ingress creation | `false` |
| `icap_ingress.class` | Sets the ingress class | `"nginx"` |
| `icap_docker_repo` |  | `"opswat"` |
| `icap_components.md_icapsrv.name` |  | `"md-icapsrv"` |
| `icap_components.md_icapsrv.image` | This value always get the image latest in the repository. Overrides the default docker image for the MD ICAP Server service, this value can be changed if you want to set a different version of MD ICAP Server (ex: opswat/metadefendericapsrv-debian:4.13.0). | `"opswat/metadefendericapsrv-debian"` |
| `icap_components.md_icapsrv.env` |  | `[{"name":"MD_USER","valueFrom":{"secretKeyRef":{"name":"mdicapsrv-cred","key":"user"}}},{"name":"MD_PWD","valueFrom":{"secretKeyRef":{"name":"mdicapsrv-cred","key":"password"}}},{"name":"APIKEY","valueFrom":{"secretKeyRef":{"name":"mdicapsrv-api-key","key":"value"}}},{"name":"LICENSE_KEY","valueFrom":{"secretKeyRef":{"name":"mdicapsrv-license-key","key":"value"}}}]` |
| `icap_components.md_icapsrv.import_configuration.enabled` |  | `"false"` |
| `icap_components.md_icapsrv.import_configuration.targets` |  | `"[schema, servers]"` |
| `icap_components.md_icapsrv.import_configuration.importConfigMap` |  | `"mdicapsrv-import-configuration"` |
| `icap_components.md_icapsrv.import_configuration.importConfigMapSubPath` |  | `"mdicapsrv-config.json"` |
| `icap_components.md_icapsrv.tls.enabled` |  | `"false"` |
| `icap_components.md_icapsrv.tls.certSecret` |  | `"mdicapsrv-tls-cert"` |
| `icap_components.md_icapsrv.tls.certSecretSubPath` |  | `"mdicapsrv.crt"` |
| `icap_components.md_icapsrv.tls.certKeySecret` |  | `"mdicapsrv-tls-cert-key"` |
| `icap_components.md_icapsrv.tls.certKeySecretSubPath` |  | `"mdicapsrv.key"` |
| `icap_components.md_icapsrv.ports` |  | `"[8048,1344,11344]"` |
| `icap_components.md_icapsrv.service_type` | Sets the service type for MD ICAP Server service (ClusterIP, NodePort, LoadBalancer) | `"ClusterIP"` |
| `icap_components.md_icapsrv.extra_labels.aws-type` | If `aws-type` is set to `fargate`, the MD ICAP Server pod will be scheduled on an AWS Fargate virtual node (if a fargate profile is provisioned and configured) | `"fargate"` |
| `icap_components.md_icapsrv.resources.requests.memory` | Minimum reserved memory | `"4Gi"` |
| `icap_components.md_icapsrv.resources.requests.cpu` | Minimum reserved cpu | `"1.0"` |
| `icap_components.md_icapsrv.resources.limits.memory` | Maximum memory limit | `"8Gi"` |
| `icap_components.md_icapsrv.resources.limits.cpu` | Maximum cpu limit | `"1.0"` |
| `icap_components.md_icapsrv.livenessProbe.httpGet.path` | Health check endpoint | `"/readyz"` |
| `icap_components.md_icapsrv.livenessProbe.httpGet.port` | Health check port | `8048` |
| `icap_components.md_icapsrv.livenessProbe.initialDelaySeconds` |  | `30` |
| `icap_components.md_icapsrv.livenessProbe.periodSeconds` |  | `10` |
| `icap_components.md_icapsrv.livenessProbe.timeoutSeconds` |  | `10` |
| `icap_components.md_icapsrv.livenessProbe.failureThreshold` |  | `3` |
| `icap_components.md_icapsrv.strategy.type` |  | `"RollingUpdate"` |
| `icap_components.md_icapsrv.strategy.rollingUpdate.maxSurge` |  | `0` |
| `icap_components.md_icapsrv.sidecars` | Configuration for the activation-manager sidecar | `[{"name": "activation-manager", "image": "alpine", "envFrom": [{"configMapRef": {"name": "mdicapsrv-env"}}], "env": [{"name": "APIKEY", "valueFrom": {"secretKeyRef": {"name": "mdicapsrv-api-key", "key": "value"}}}, {"name": "LICENSE_KEY", "valueFrom": {"secretKeyRef": {"name": "mdicapsrv-license-key", "key": "value"}}}], "command": ["/bin/sh", "-c"], "args": ["apk add curl jq\nstop() {\n  echo 'Deactivating using the MD ICAP Server API'\n  curl -H \"apikey: $APIKEY\" -X POST \"https://localhost:$REST_PORT/admin/license/deactivation\"\n  echo 'Deactivating using activation server API'\n  curl -X GET \"https://$ACTIVATION_SERVER/deactivation?key=$LICENSE_KEY&deployment=$DEPLOYMENT\"\n  exit 0\n}\ntrap stop SIGTERM SIGINT SIGQUIT\n\nuntil [ -n $DEPLOYMENT ] && [ $DEPLOYMENT != null ]; do\n    echo 'Checking...'\n    export DEPLOYMENT=$(curl --silent -H \"apikey: $APIKEY\" \"http://localhost:$REST_PORT/admin/license\" | jq -r \".deployment\")\n    echo \"Deployment ID: $DEPLOYMENT\"\n    sleep 1\ndone\necho \"Waiting for termination signal...\"\nwhile true; do sleep 1; done\necho \"MD ICAP Server pod finished, exiting\"\nexit 0\n"]}]` |
| `nodeSelector` |  | `{}` |

## Notice
- Set "import_configuration" to false if you do not have file "mdicapsrv-config.json" in "/opt/opswat/mdicapsrv-config.json". By this option, after finish installation you must config MD ICAP Server manually or import an existing configuration by import/export feature.
- To have a file "mdicapsrv-config.json" correctly, please install a MD ICAP Server, do configuration setting then use export feature to get the json config file.
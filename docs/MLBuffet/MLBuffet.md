# MLBuffet

### Index
- [Introduction](#introduction)
- [Documentation](#documentation)
- [Pre-requisites](#pre-requisites)
- [Install & Configure MLBuffet](#install-and-configure-mlbuffet)
- [Using MLBuffet](#using-mlbuffet)

### Introduction
[MLBuffet](https://github.com/zylklab/mlbuffet) is a Machine Learning Model Server based on containers. 
MLBuffet consists of several modules:
- Deploy module: Contains the necessary files to install the project, Docker Swarm or Kubernetes.
- Inferrer module: The main REST API. Users or clients may communicate with this API to access the app resources.
- Modelhost module: Workers for model deployments and inference. There might be multiple instances of this module, each being aware of all the models stored.
- Metrics Gathers and manages performance metrics from host system and services.
- Storage module: Performs version controlling.

### Documentation
- [MLBuffet](https://github.com/zylklab/mlbuffet): Zylk´s Github repository.

### Pre-requisites
- Access to a container registry on which the docker images containing the different models that compose
MLBuffet have been pushed.

### Install and Configure MLBuffet
For installing MLBuffet a [deploy.sh](../../deployment/MLBuffet/deploy.sh) script will be used. This script applies the YAML files 
under the autodeploy directory. However, before applying the YAMLs that install and configure
MLBuffet, some resources must be configured in through 
the [kustomization.yaml](../../deployment/MLBuffet/autodeploy/kustomization.yaml) and [inferrer.yaml](../../deployment/MLBuffet/autodeploy/inferrer.yaml) files. 
In this YAML the docker images that will be used for the deployment of MLBuffet must be specified:


[inferrer.yaml](../../deployment/MLBuffet/autodeploy/inferrer.yaml)

```yaml
spec:
  serviceAccountName: modelhost-creator  
  containers:
    - name: inferrer
      image: vtwkpdos.gra7.container-registry.ovh.net/fractal/mlbuffet_inferrer:v2 
      imagePullPolicy: Always
      ports:
        - containerPort: 8000
      env:
        - name: TRAINER_MANAGER
          value: KUBERNETES
        - name: IMAGE_MLBUFFET_TRAINER
          value: vtwkpdos.gra7.container-registry.ovh.net/fractal/mlbuffet_trainer:v2
        - name: IMAGE_MLBUFFET_MODELHOST
          value: vtwkpdos.gra7.container-registry.ovh.net/fractal/mlbuffet_modelhost:v2
  imagePullSecrets:
    - name: fractalregistry
---
```

[kustomization.yaml](../../deployment/MLBuffet/autodeploy/kustomization.yaml)

```yaml
images:
  # Inferrer image name
  - name: IMAGE_MLBUFFET_INFERRER
    newName: vtwkpdos.gra7.container-registry.ovh.net/fractal/mlbuffet_inferrer:v2
  # Cache image name
  - name: IMAGE_MLBUFFET_CACHE
    newName: vtwkpdos.gra7.container-registry.ovh.net/fractal/mlbuffet_cache:v2
  # Metrics image name
  - name: IMAGE_MLBUFFET_METRICS
    newName: vtwkpdos.gra7.container-registry.ovh.net/fractal/mlbuffet_metrics:v2
  # Storage image name
  - name: IMAGE_MLBUFFET_STORAGE
    newName: vtwkpdos.gra7.container-registry.ovh.net/fractal/mlbuffet_storage:v2
```

Afterwards, the [deploy.sh](../../deployment/MLBuffet/deploy.sh) script must be executed:
```bash
./deploy.sh
```


#### Exposing MLBuffet
For exposing MLBuffet, the [ingress-mlbuffet.yml](../../deployment/MLBuffet/ingress-mlbuffet.yml) file will be applied:

```bash
kubectl apply ingress-mlbuffet.yml
```

### Using MLBuffet
MLBuffet API will be exposed at https://fractal.ik-europe.eu/mlbuffet/api/v1/. To test that MLBuffet has been
deployed properly, the following endpoint can be accessed:


```bash
curl -k https://fractal.ik-europe.eu/mlbuffet/
#response
{
    "http_status": {
        "code": 200,
        "description": "Greetings from MLBuffet - Inferrer, the Machine Learning model server. For more information, visit /help",
        "name": "OK"
    }
}
```
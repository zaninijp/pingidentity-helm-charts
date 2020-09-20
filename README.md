# Sample Ping Identity Helm Charts - Solutions

This repository is a fork that provides sample Helm charts for Ping Identity - [Customer360](https://github.com/pingidentity/Customer360) Solutions for community use.

Product charts have been modified to use a Global values file that is mapped across all the deployed stack.

The Customer360 chart will create a full set of Deployments \ Services \ ConfigMaps \ Secrets \ Ingress files that allow everything within a Release to work together. PingAccess is included in this to enable simplified client-routing, and ACME support if desired.

These helm charts are not in a finished or production-ready state, but are intended to be a good starting point and can be used and altered as required.

Note: These charts are development / POC quality and originally put together as a learning experience.  Pull requests and issues welcome but don't expect a quick response!

## Charts - Solution
| Chart | Type | Capability | Status | Scalable |
|--|--|--|--|--|
| [Customer360](customer360/) | Solution | CIAM Data Store / CIAM SSO / CIAM Authentication Authority / CIAM MFA | Available (Beta) | N/A |

## Charts - Use Cases
| Chart | Type | Capability | Status | Scalable |
|--|--|--|--|--|
| [Consent Management](consentmanagement/) | Use Case | Consent API configuration | Available (Beta) | N/A |

## Charts - Individual Products
| Chart | Type | Capability | Status | Scalable |
|--|--|--|--|--|
| [PingFederate](pingfederate/) | Product | SSO / Authentication Authority | Available (Beta) | Yes |
| [PingAccess](pingaccess/) | Product | Medium/Course grain Authorisation Gateway / PEP | Available (Beta) | No (TBC) |
| [PingDirectory](pingdirectory/) | Product | User/Device/Consent/Organisation Directory | Available (Beta) | Yes |
| [PingDelegator](pingdelegator/) | Product | Delegated User Management UI | Available (Beta) | Yes |
| [PingCentral](pingcentral/) | Product | Delegated Application Management UI | Available (Beta) | Yes |
| [PingDataConsole](pingdataconsole/) | Product | PingData Admin Console | Available (Beta) | Not required |
| [PingDataSync](pingdatasync/) | Product | Data Synchronisation Engine | Available | N/a |
| PingDataGovernance | Product | Fine-grain Authorisation PDP | Not available | N/a |
| PingDataGovernance PAP | Product | Fine-grain Authz PAP | Not available | N/a |
| PingDirectoryProxy | Product | Directory Proxy | Not available | N/a |

## Add the Repository

```shell
helm repo add pingidentity-solutions https://cprice-ping.github.io/pingidentity-solutions-helm/
helm repo update
helm repo list
```

Search for a Ping chart:

```shell
helm search repo ping
```

## Use OCI Support

Helm v3.x has experimental support for Container Registries - allowing for simpler retrieval of charts. To use this:

1. Enable OCI Support:

  ```shell
  export HELM_EXPERIMENTAL_OCI=1
  ```

2. Retrieve the Customer360 chart

  ```shell
  helm chart export pingsolutions.azurecr.io/helm/customer360:latest .
  ```

3. Deploy the chart with a named deployment - `releaseName`

  ```shell
  helm upgrade --install {{releaseName}} ./customer360/ -f values.yaml
  ```

4. Examine Deployment

  Use `kubectl` to see everything that Helm deployed - all components will be prefixed with {{releaseName}}

  ```shell
laptop$ kubectl get services
NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                            AGE
cpricelab-mysql                ClusterIP   172.20.49.47     <none>        3306/TCP                           3h25m
cpricelab-mysql-slave          ClusterIP   172.20.128.8     <none>        3306/TCP                           3h25m
cpricelab-pingaccess           ClusterIP   172.20.239.172   <none>        9000/TCP,3000/TCP,8080/TCP         3h25m
cpricelab-pingcentral          ClusterIP   172.20.56.182    <none>        9022/TCP                           3h25m
cpricelab-pingdataconsole      ClusterIP   172.20.138.30    <none>        8443/TCP                           3h25m
cpricelab-pingdatasync         ClusterIP   172.20.23.59     <none>        443/TCP,636/TCP,389/TCP            3h25m
cpricelab-pingdirectory        ClusterIP   172.20.186.251   <none>        443/TCP,636/TCP,389/TCP,2443/TCP   3h25m
cpricelab-pingfederate         ClusterIP   172.20.26.68     <none>        443/TCP,9031/TCP                   3h25m
```

5. See all your Helm Releases -- `helm list`

```shell
laptop$ helm list
NAME            NAMESPACE                   REVISION  UPDATED                             STATUS  CHART                  APP VERSION
cpricelab         ping-cloud-devops-eks-cprice  1         2020-09-20 12:32:24.957381 -0700 PD  deployed  customer360-0.0.42     2.0.0
cpricelab-consent ping-cloud-devops-eks-cprice  1       2020-09-19 11:00:47.625663 -0700 PDT	deployed	consentmanagement-0.0.2	1.0.0
```

## Injecting Ping Product Variables

You can inject your own environment variables to modify the behaviour of the Ping Docker images, or when using environment variables in your own server profiles.  

The bulk of the environment is pre-configured to work with the PingIdentity [Customer360 Profile](https://github.com/pingidentity/Customer360) - to add your specific information to that deployment, you need to create a `values.yaml` file (see [Sample](./values-sample.yaml) ).

Additionaly, Product-Specific values can be overridden in this file too:

```
pingfederate:
  pingfederate:
    envs:
      SERVER_PROFILE_URL: https://github.com/myuser/mycustomrepo.git 
      SERVER_PROFILE_PATH: my/path/to/pingfederate
      SERVER_PROFILE_BRANCH: master
      MY_SERVER_PROFILE_VAR: "your value here"
```

## Waiting for dependent services

For an example of waiting for a dependent service, such as PingDirectory, see the example [here](https://github.com/patrickcping/ping-helm-kustomize-example)

## Adding Use Cases
You can use Helm to add additional configs to Customer360 - this may be another API collection, or additional Products.

While these are seen as a separate deployment, the Charts can be pointed to the proper C360 Release with the `--set releaseName={{releaseName}}` flag.

For example:

```shell
helm chart export pingsolutions.azurecr.io/helm/consentmanagement:latest .
helm install cpricelab-consent ./consentmanagement/ --set releaseName=cpricelab -f values.yaml
```
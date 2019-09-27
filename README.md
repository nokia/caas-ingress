# caas-ingress

[![Actions Status](https://github.com/nokia/caas-ingress/workflows/CI/badge.svg)](https://github.com/nokia/caas-ingress/actions)

## Introduction

`caas-ingress` is a helm chart to expose [Akraino](https://www.lfedge.org/projects/akraino/) [Radio Edge Cloud](https://wiki.akraino.org/pages/viewpage.action?pageId=6128402) (REC) internal CaaS services over external interfaces, so external monitoring applications can access them over HTTPS protocol.

**Disclaimer: This is not an official part of REC or RIC product. Since it is an individual opensource software, You have the responsibility to decide how to use it properly to avoid any security flaws in your infrastructure.**

The following CaaS services are proxied externally:
<table>
  <tr><th>CaaS component name</th><th>Internal URL</th><th>External URL</th></tr>
  <tr><td>Elasticsearch</td><td>http://elasticsearch-logging.kube-system.svc.rec.io:9200</td><td>https://any_controller_ext_ip:19200</td></tr>
  <tr><td>Prometheus</td><td>https://prometheus.kube-system.svc.rec.io:9090</td><td>https://any_controller_ext_ip:19090</td></tr>
  <tr><td>Kubernetes API server</td><td>https://10.254.0.1:443</td><td>https://any_controller_ext_ip:16443</td></tr>
</table>

The external IP addresses and ports are configurable in the `values.yaml` file.

For external HTTPS traffic an option is provided to use a custom TLS certificate (preferably signed by a trusted CA). This certificate must be loaded into a TLS Secret inside the same namespace, where the chart will be deployed into.
If custom TLS Secret is not specified, a new self-signed TLS key is generated during chart deployment. This means the external tools will complain about the untrusted key.

For K8S API access a restricted ServiceAccount is being used, which has `get` & `list` permissions for the following endpoints:

* `/api/v1/nodes`
* `/api/v1/pods`
* `/api/v1/namespaces`
* `/api/v1/nodes/<nodename>`
* `/api/v1/namespaces/keyhole`
* `/api/v1/namespaces/kube-system/pods/<podname>`
* `/apis/metrics.k8s.io/v1beta1/nodes`
* `/apis/metrics.k8s.io/v1beta1/nodes/<nodename>`
* `/apis/metrics.k8s.io/v1beta1/pods`
* `/apis/metrics.k8s.io/v1beta1/namespaces/kube-system/pods/<podname>`
* `/apis/custom.metrics.k8s.io/v1beta1`

## How to deploy

1. Download and unpack the latest `caas-ingress` release.

2. Create a namespace for it:

   ```sh
   kubectl create namespace caas-ingress
   ```

3. Create `caas-ingress-sa` ServiceAccount for restricted K8S API access:

   ```sh
   kubectl create -f caas-ingress/caas-ingress-sa.yaml
   ```

4. OPTIONAL: Load your custom TLS certification (named as `tls.crt` & `tls.key`) into a TLS Secret:

   ```sh
   kubectl -n caas-ingress create secret tls caas-ingress-https --cert=tls.crt --key=tls.key
   ```

5. Deploy the helm chart:

   ```sh
   helm install -n caas-ingress caas-ingress*/ --namespace caas-ingress --set service.externalIPs="{10.20.30.41,10.20.30.42,10.20.30.43}" --wait
   ```

   or in case custom TLS certification is provided:

   ```sh
   helm install -n caas-ingress caas-ingress*/ --namespace caas-ingress --set service.externalIPs="{10.20.30.41,10.20.30.42,10.20.30.43}" --set custom_tls_secret=caas-ingress-https --wait
   ```

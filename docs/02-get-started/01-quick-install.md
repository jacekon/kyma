# Quick Install

> [!WARNING]
> This guide describes the installation of standalone open-source Kyma with specific modules. If you are a managed offering user of **SAP BTP, Kyma runtime**, read [Add and Delete a Kyma Module](https://help.sap.com/docs/btp/sap-business-technology-platform/enable-and-disable-kyma-module?locale=en-US&version=Cloud) instead.

To get started with Kyma, let's quickly install it with specific modules first.

## Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- Kubernetes cluster, or for local installation: [k3d](https://k3d.io) (v5.x or higher) as well as [Docker](https://www.docker.com/)
- `kyma-system` namespace created
- [Kyma CLI](https://github.com/kyma-project/cli)

## Steps

1. To provision a k3d cluster, run:

  ```bash
  k3d cluster create kyma --kubeconfig-switch-context -p '30080:80@loadbalancer' -p '30443:443@loadbalancer' --k3s-arg '--disable=traefik@server:*' --k3s-arg '--tls-san=host.docker.internal@server:*'
  kubectl create ns kyma-system
  ```

2. To install a Kyma module of your choice on a Kubernetes cluster, deploy its module manager and apply the module configuration. See the following Kyma modules with their quick installation commands and links to their GitHub repositories:

  [**Istio**](https://github.com/kyma-project/istio)

  ```bash
  kubectl label namespace kyma-system istio-injection=enabled --overwrite
  kubectl apply -f https://github.com/kyma-project/istio/releases/latest/download/istio-manager.yaml
  kubectl apply -f https://github.com/kyma-project/istio/releases/latest/download/istio-default-cr.yaml
  ```
  
  [**SAP BTP Operator**](https://github.com/kyma-project/btp-manager)

  ```bash
  kubectl apply -f https://github.com/kyma-project/btp-manager/releases/latest/download/btp-manager.yaml
  kubectl apply -f https://github.com/kyma-project/btp-manager/releases/latest/download/btp-operator-default-cr.yaml -n kyma-system
  ```

  > [!WARNING]
  > The CR is in the `Warning` state and the message is `Secret resource not found reason: MissingSecret`. To create the Secret, follow the instructions in [Create `sap-btp-manager` Secret](https://github.com/kyma-project/btp-manager/blob/main/docs/user/03-00-create-btp-manager-secret.md).

  [**Application Connector**](https://github.com/kyma-project/application-connector-manager)

  ```bash
  kubectl apply -f https://github.com/kyma-project/application-connector-manager/releases/latest/download/application-connector-manager.yaml
  kubectl apply -f https://github.com/kyma-project/application-connector-manager/releases/latest/download/default_application_connector_cr.yaml -n kyma-system
  ```

  [**Keda**](https://github.com/kyma-project/keda-manager)

  ```bash
  kubectl apply -f https://github.com/kyma-project/keda-manager/releases/latest/download/keda-manager.yaml
  kubectl apply -f https://github.com/kyma-project/keda-manager/releases/latest/download/keda-default-cr.yaml -n kyma-system
  ```

  [**Serverless**](https://github.com/kyma-project/serverless-manager)

  ```bash
  kubectl apply -f https://github.com/kyma-project/serverless-manager/releases/latest/download/serverless-operator.yaml
  kubectl apply -f https://github.com/kyma-project/serverless-manager/releases/latest/download/default-serverless-cr.yaml  -n kyma-system
  ```

  [**Telemetry**](https://github.com/kyma-project/telemetry-manager)

  ```bash
  kubectl apply -f https://github.com/kyma-project/telemetry-manager/releases/latest/download/telemetry-manager.yaml
  kubectl apply -f https://github.com/kyma-project/telemetry-manager/releases/latest/download/telemetry-default-cr.yaml -n kyma-system
  ```

  [**NATS**](https://github.com/kyma-project/nats-manager)

  ```bash
  kubectl apply -f https://github.com/kyma-project/nats-manager/releases/latest/download/nats-manager.yaml
  kubectl apply -f https://github.com/kyma-project/nats-manager/releases/latest/download/nats-default-cr.yaml -n kyma-system
  ```

  [**Eventing**](https://github.com/kyma-project/eventing-manager)

  ```bash
  kubectl apply -f https://github.com/kyma-project/eventing-manager/releases/latest/download/eventing-manager.yaml
  kubectl apply -f https://github.com/kyma-project/eventing-manager/releases/latest/download/eventing-default-cr.yaml -n kyma-system
  ```

  [**API Gateway**](https://github.com/kyma-project/api-gateway)

  ```bash
  kubectl label namespace kyma-system istio-injection=enabled --overwrite
  kubectl apply -f https://github.com/kyma-project/api-gateway/releases/latest/download/api-gateway-manager.yaml
  kubectl apply -f https://github.com/kyma-project/api-gateway/releases/latest/download/apigateway-default-cr.yaml
  ```
  
3. Update CoreDNS to correctly resolve the `local.kyma.dev` domain:

  ```bash
  cat <<EOF | kubectl apply -f -
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: coredns-custom
    namespace: kube-system
  data:
    kyma.override: |
      rewrite name regex (.*)\.local\.kyma\.dev istio-ingressgateway.istio-system.svc.cluster.local
  EOF

  kubectl rollout restart deployment -n kube-system coredns
  ```

4. To manage Kyma using graphical user interface (GUI), open Kyma dashboard:

  ```bash
  kyma dashboard
  ```
  <!-- markdown-link-check-disable-next-line -->
  This command takes you to your Kyma dashboard under [`http://localhost:3001/`](http://localhost:3001/).

## Related Links

- To see the list of all available Kyma modules, go to [Kyma modules](../06-modules/README.md).
- To learn how to [uninstall and upgrade Kyma with a module](./08-uninstall-upgrade-kyma-module.md).

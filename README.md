# gloo-edge-team-gateway

Guide to install Gloo Edge control plane in one namespace and the gateway in a different namespace:
1- Install  gloo edge, in this example I'm installing it in gloo-system, it is not possible today to install everything but the default gateway using the current helm chart, I'll create a ticket to has this feature, but in the meantime you can scale down the gateway-proxy deployment to 0, or use Kustomize
2-  edit the gloo edge setting to set this value:
```   
    gateway:
      readGatewaysFromAllNamespaces: true 
      ```
by default it is set to false
3- create the a different namespace for your team, in this example I call it alpha, then apply the attached manifest,
```
kubectl apply -f gateway-alpha-full.yaml -n alpha
```
couple notes:
the manifest contains: a deployment and a service for your gateway, 2 gateway crd, where you can define your gatway configuration (listener) and a config map what contains the bootstrap configuration for envoy, the most important in that configmap is the role: "gloo-system~gateway-proxy-team-alpha", so that the configuration pulled from the gloo xds service is related to this gateway.
To test the configuration, create an upstream and a virtual service, for example
```
glooctl -n alpha create upstream static --name echo --static-hosts postman-echo.com:80
glooctl add route  \                                                                                                                                                                                
    --name default  \
    --path-prefix / \
    --dest-name echo --namespace alpha -n alpha
this will add a upstream and a virtualservice in the alpha namespace, then the gateway will select it. then you can test your gateway config:
kubectl port-forward -n alpha svc/gateway-proxy-team-alpha 8080:80
curl localhost:8080/request                                                                                                                                                                         
{"args":{},"headers":{"x-forwarded-proto":"http","x-forwarded-port":"80","host":"localhost","x-amzn-trace-id":"Root=1-6089dece-5fa77d1663d9261f2c775d22","user-agent":"curl/7.64.1","accept":"*/*","x-request-id":"24c64205-02e0-48af-8361-3a675d0b8300","x-envoy-expected-rq-timeout-ms":"15000"},"url":"http://localhost/request"}```

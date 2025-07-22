# Improvements Made to Kubernetes Manifests

## 1. Ingress Resource Modernization
- **Replaced deprecated annotation** `kubernetes.io/ingress.class: alb` with the recommended `ingressClassName: alb` field in the Ingress spec.
- This aligns with AWS and Kubernetes best practices for clusters running Kubernetes 1.18 and above.
- The manifest now looks like:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
    # kubernetes.io/ingress.class: alb <=== moved from here
spec:
  ingressClassName: alb     # put here
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: react-ui-service
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: spring-boot-service
                port:
                  number: 8080
```

---
  
## 2. Explicit IngressClass Resource for ALB
- Created a dedicated `IngressClass` resource named `alb` in `alb-ingressclass.yml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: alb
spec:
  controller: ingress.k8s.aws/alb
```
- We can make it **default** by setting
```yaml
metadata:
    annotations:
        ingressclass.kubernetes.io/is-default-class: "true"
```

  - The Ingress manifest now references this class using `ingressClassName: alb`, ensuring explicit and modern configuration for AWS ALB Ingress Controller.

---

## 3. Custom Domain and SSL with AWS ALB Ingress
- Configured the Ingress to use the custom domain `amazoneks.dockeroncloud.com` by adding a `host` entry in the rules.
- Enabled HTTPS by specifying the ACM certificate ARN with the annotation `alb.ingress.kubernetes.io/certificate-arn`.
- Configured the ALB to listen on both HTTP (80) and HTTPS (443), and redirect HTTP to HTTPS using `alb.ingress.kubernetes.io/ssl-redirect: '443'`.
- This setup ensures secure access to your application using your custom domain and SSL certificate.

Example manifest changes:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/certificate-arn: <YOUR_ACM_CERTIFICATE_ARN>
    alb.ingress.kubernetes.io/ssl-redirect: '443'
spec:
  ingressClassName: alb
  rules:
    - host: amazoneks.dockeroncloud.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: react-ui-service
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: spring-boot-service
                port:
                  number: 8080
```

**Note:** Replace `<YOUR_ACM_CERTIFICATE_ARN>` with your actual ACM certificate ARN for amazoneks.dockeroncloud.com.

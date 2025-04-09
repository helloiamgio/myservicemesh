# Istio Service Mesh - Guida Completa

Benvenuto in questa guida completa a **Istio Service Mesh**, pensata come README per un repository Git. Qui troverai una panoramica tecnica di Istio, i principali oggetti API con esempi pratici e l'integrazione con **Kiali** per il monitoraggio.

---

## 📌 Cos'è Istio

**Istio** è una **service mesh** open-source progettata per facilitare la gestione, il monitoraggio e la sicurezza della comunicazione tra microservizi. Istio si integra con Kubernetes e fornisce:

- **Traffic management** (routing, load balancing, failover)
- **Sicurezza** (mTLS, autorizzazione)
- **Observability** (telemetria, tracing, logging)

Istio introduce un **data plane** (proxy Envoy in sidecar) e un **control plane** (gestito da `istiod`).

[Scarica il documento RH-SM (PDF)](RH-SM.pdf)

---

## 🏗️ Architettura

```text
            +----------------+
            |     Kiali      |<-------------------+
            +----------------+                    |
                     |                            |
                     v                            |
            +----------------+         +----------------+
            |    Istiod      |<------->|  Prometheus     |
            +----------------+         +----------------+
                     |
     +---------------+--------------+
     |                              |
     v                              v
+-----------+                 +-----------+
|   Service | <--> Sidecar <->|   Service |
+-----------+     (Envoy)     +-----------+
```

![Diagramma Service Mesh](SH-SM.jpg)

---

## 📘 Oggetti API principali di Istio (con esempi)

### 1. VirtualService

Definisce **come** il traffico viene instradato verso i servizi.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews-routing
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

### 2. DestinationRule

Definisce le **policy di traffico** per un servizio.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
```

### 3. Gateway

Espone i servizi al traffico **esterno** al mesh.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: my-ingress-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

### 4. ServiceEntry

Permette l'accesso verso **servizi esterni** al mesh.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: allow-google
spec:
  hosts:
  - www.google.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```

### 5. PeerAuthentication

Definisce policy di **autenticazione mTLS** tra sidecar.

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
```

### 6. AuthorizationPolicy

Definisce **chi può accedere a cosa** nel mesh.

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-admins-only
spec:
  selector:
    matchLabels:
      app: productpage
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/admin"]
```

---

## 🧩 Installazione di Istio (demo profile)

```bash
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
```

---

## 🧪 Installazione e utilizzo di Kiali

### Installazione (addon ufficiale):

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml
```

### Accesso:

```bash
kubectl port-forward svc/kiali 20001:20001 -n istio-system
# Poi apri: http://localhost:20001
```

### Funzionalità principali:

- **Graph View**: visualizza la topologia e il traffico tra i servizi.
- **Traffic Flow**: flusso real-time.
- **Istio Config**: visione centralizzata di tutti gli oggetti (VS, DR, GW).
- **Metrics View**: metriche integrate via Prometheus.
- **Traces**: integrazione con Jaeger per tracing distribuito.

---

## ✅ Requisiti

- Kubernetes >= 1.24
- Helm (opzionale)
- Istioctl installato

---

## 📁 Struttura del repository suggerita

```
service-mesh/
├── README.md
├── istio-config/
│   ├── virtualservice.yaml
│   ├── destinationrule.yaml
│   ├── gateway.yaml
│   ├── serviceentry.yaml
│   ├── peerauthentication.yaml
│   └── authorizationpolicy.yaml
└── kiali/
    └── kiali-install.yaml
```

---

## 🙌 Contribuire

Pull request e issue sono benvenuti per estendere questa guida o aggiungere casi d'uso avanzati!

---

## 📜 Licenza

Questo progetto è distribuito sotto licenza MIT.

---

## 🔗 Risorse utili

- [Istio Docs](https://istio.io/latest/docs/)
- [Kiali.io](https://www.kiali.io/)
- [Jaeger Tracing](https://www.jaegertracing.io/)

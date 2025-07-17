# AWS Elastic Load Balancer (ELB) Tutorial

## Ce este ELB?
Elastic Load Balancer (ELB) distribuie automat traficul de rețea sau aplicație pe mai multe instanțe Amazon EC2, containere sau adrese IP. ELB ajută la asigurarea disponibilității și scalabilității aplicațiilor.

---

## Tipuri de Load Balancer în AWS

### 1. **Application Load Balancer (ALB)**
- **Layer**: Layer 7 (Aplicație - HTTP/HTTPS)
- **Cazuri de utilizare**:
  - Routing bazat pe conținut (URL-uri, host headers, etc.)
  - Web applications și microservicii
  - Integrare cu container orchestration (ECS, EKS)
- **Caracteristici**:
  - Suportă WebSocket și HTTP/2
  - Path-based și host-based routing
  - Redirecturi și returnare coduri personalizate

---

### 2. **Network Load Balancer (NLB)**
- **Layer**: Layer 4 (Transport - TCP/UDP/TLS)
- **Cazuri de utilizare**:
  - Aplicații cu performanță ridicată și latență redusă
  - Protocol TCP, UDP sau TLS
  - Static IP și suport pentru Elastic IP
- **Caracteristici**:
  - Scalare automată
  - Connection-based routing (nu inspectează layer 7)
  - Suport pentru TLS termination

---

### 3. **Gateway Load Balancer (GWLB)**
- **Layer**: Layer 3/4 (IP + Transport)
- **Cazuri de utilizare**:
  - Integrarea appliance-urilor de rețea (firewall, DPI, IDS/IPS)
  - Service chaining și middlebox virtualization
- **Caracteristici**:
  - Folosește GENEVE encapsulation
  - Interoperabil cu third-party appliances
  - Transparent traffic forwarding

---

## Comparativ rapid

| Caracteristică         | ALB                  | NLB                  | GWLB                 |
|------------------------|----------------------|----------------------|----------------------|
| Nivel OSI              | Layer 7              | Layer 4              | Layer 3/4            |
| Protocole              | HTTP, HTTPS          | TCP, UDP, TLS        | IP (GENEVE)          |
| Routare bazată pe conținut | ✔️                  | ❌                   | ❌                   |
| Static IP support      | ❌ (via NLB only)     | ✔️                   | ✔️                   |
| Latență foarte mică    | ❌                   | ✔️                   | ✔️                   |
| Integrare cu ECS/EKS   | ✔️                   | ✔️                   | ❌                   |

---

## Alte informații utile

- Toate tipurile de ELB sunt _managed services_, deci AWS se ocupă de:
  - Health checks
  - Auto-scaling
  - Fault tolerance
- Pot fi folosite cu Auto Scaling Groups
- Logging disponibil prin Access Logs (pentru ALB și NLB)
- Integrare cu AWS WAF (doar pentru ALB)

---

## Concluzie

- **ALB**: Ideal pentru aplicații web și microservicii care necesită routing complex.
- **NLB**: Ideal pentru aplicații critice cu latență scăzută și conexiuni rapide.
- **GWLB**: Ideal pentru scenarii avansate de rețea (ex: inspecție trafic, firewall virtualizat).

---

## Exemple de arhitectură

- Frontend SPA → ALB → ECS cu Fargate
- IoT UDP traffic → NLB → EC2 cu agent custom
- Trafic filtrat → GWLB → Appliance → Backend


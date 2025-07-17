# AWS Application Load Balancer (ALB) – Ghid complet

## 🔷 Ce este ALB?

Application Load Balancer (ALB) este un serviciu AWS Elastic Load Balancing care funcționează la nivelul **Layer 7 (HTTP/HTTPS)** din modelul OSI. ALB este folosit pentru a distribui inteligent traficul web către mai multe servere (EC2, ECS, EKS etc.) în funcție de reguli avansate precum path, hostname, headere sau query strings.

---

## 📌 Când se folosește?

ALB este recomandat când:

- Ai aplicații web REST sau microservicii
- Vrei routing pe bază de path/host (ex: `/api`, `admin.site.com`)
- Ai nevoie de WebSocket, HTTP/2, TLS termination
- Folosești ECS, EKS sau Lambda
- Ai nevoie de autentificare, redirecturi sau cookie-based stickiness

---

## 🔑 Functionalități cheie

- **Host-based routing**: `api.example.com` → service A
- **Path-based routing**: `/admin/*` → service B
- **Header/cookie/query-based routing**
- **Redirect & fixed-response rules**
- **WebSocket support**
- **HTTP/2 support**
- **HTTPS (TLS Termination)** cu certificate ACM
- **Sticky sessions (session affinity)**
- **Integration cu ECS, EKS (Ingress), Lambda**

---

## ⚙️ Componente ALB

1. **Load Balancer**
2. **Listeners** (ex: 80, 443)
3. **Listener Rules** (condiții de routing)
4. **Target Groups** (EC2, ECS, IPs, Lambda)
5. **Health Checks** (HTTP path-uri, port)

---

## 🔐 Certificate SSL/TLS

Dacă vrei HTTPS:

- Creează un certificat în **AWS Certificate Manager (ACM)**
- Validează domeniul (DNS recomandat)
- Atașează certificatul la Listener-ul de pe portul 443
- Activează redirect de la HTTP → HTTPS

---

## 🔧 Optimizări și Tweaks

| Setare                   | Detalii |
|--------------------------|---------|
| **Sticky Sessions**      | `lb_cookie` stickiness pe target group |
| **Slow Start**           | Trimite treptat trafic către noi instanțe |
| **Connection Timeout**   | Implicit 60s (poate fi ajustat) |
| **Cross-Zone LB**        | Activează load balancing între AZ-uri |
| **Access Logs**          | Trimite în S3 pentru analiză traficului |
| **WAF**                  | Poate fi atașat direct la ALB |

---

## 📊 Monitorizare (CloudWatch)

| Metrică CloudWatch             | Semnificație                          |
|--------------------------------|---------------------------------------|
| `RequestCount`                 | Total requesturi                      |
| `TargetResponseTime`           | Timp răspuns mediu backend            |
| `HTTPCode_Target_5XX_Count`    | Erori 5XX backend                     |
| `HealthyHostCount`             | Număr instanțe sănătoase              |

---

## 🛠️ SCENARIU REAL: Utilizator delogat constant

### 🔴 Problemă:
Un utilizator se loghează pe aplicație → dar este delogat frecvent sau pierde sesiunea.

### 🧩 Cauze posibile în ALB:

#### 1. **Lipsă Sticky Sessions**
ALB trimite requesturile în mod round-robin către instanțe diferite → sesiunea există doar pe instanța inițială → userul apare delogat.

✅ **Soluție**:
Activează Sticky Sessions în target group:
```hcl
stickiness {
  type            = "lb_cookie"
  enabled         = true
  cookie_duration = 3600 # secunde
}
```

#### 2. **Sesiune stocată local în instanță**
Dacă aplicația stochează sesiunea în memorie locală (ex: în `HttpSession` în Java), nu poate fi partajată între instanțe.

✅ **Soluție**:
Stochează sesiunea în Redis, RDS sau un alt backend centralizat.

#### 3. **Expirarea rapidă a cookie-ului ALB**
Cookie-ul `AWSALB` poate expira înainte de finalizarea sesiunii utilizatorului.

✅ **Soluție**:
Mărește durata: `cookie_duration = 86400` (1 zi)

#### 4. **Probleme cu Secure/SameSite pe cookie-uri**
Dacă folosești TLS termination în ALB, dar aplicația are cookie-uri cu flaguri stricte (`Secure`, `SameSite=Strict`), acestea pot să nu se salveze corect în browser.

✅ **Soluție**:
Verifică și ajustează setările cookie-urilor în backend.

#### 5. **Health check prea agresiv**
Instanțele pot fi eliminate frecvent de ALB dacă health check-ul returnează 5xx din greșeală → requestul e rerutat către alt backend.

✅ **Soluție**:
Configurează health check pe un endpoint light (`/health`), setează `interval=30s`, `unhealthy_threshold=3`

---

## 🧪 Exemplu: Activare sticky sessions

```hcl
resource "aws_lb_target_group" "my_app" {
  name     = "app-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  stickiness {
    type            = "lb_cookie"
    enabled         = true
    cookie_duration = 3600
  }

  health_check {
    path                = "/health"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 3
    unhealthy_threshold = 2
    matcher             = "200"
  }
}
```

---

## 📦 Costuri

- Se plătește:
  - Per oră pentru ALB activ
  - Per **LCU (Load Balancer Capacity Unit)**  
    1 LCU ≈ 25Mbps, 3000 requesturi/sec sau 1M conexiuni active

---

## 🧩 Recomandări DevOps

- Folosește ALB pentru EKS Ingress Controller (AWS Load Balancer Controller)
- Activează stickiness DOAR dacă nu ai sesiuni partajate
- Ține `health check` endpointul rapid și constant
- Activează access logs pentru debugging

---

## 📘 Documentație AWS

- [ALB – AWS Official Docs](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [ACM Certificate Setup](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request.html)
- [ALB Ingress Controller (EKS)](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)

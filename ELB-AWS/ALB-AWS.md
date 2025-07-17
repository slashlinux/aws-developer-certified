# AWS Application Load Balancer (ALB) – Ghid complet
<img width="1119" height="531" alt="Screenshot 2025-07-17 at 12 43 41" src="https://github.com/user-attachments/assets/4abbb7d3-c9a7-4b5e-97c3-4d5fd231e332" />



## 🔷 Ce este ALB?

Application Load Balancer (ALB) este un serviciu AWS Elastic Load Balancing care funcționează la nivelul **Layer 7 (HTTP/HTTPS)** din modelul OSI. ALB este folosit pentru a distribui inteligent traficul web către mai multe servere (EC2, ECS, EKS etc.) în funcție de reguli avansate precum path, hostname, headere sau query strings.

---

## 📌 Când se folosește?

ALB este recomandat când:
<img width="1115" height="537" alt="Screenshot 2025-07-17 at 12 44 57" src="https://github.com/user-attachments/assets/d09786ab-5df0-4586-ada9-5d22d14db2b0" />


- Ai aplicații web REST sau microservicii
- Vrei routing pe bază de path/host (ex: `/api`, `admin.site.com`)
- Ai nevoie de WebSocket, HTTP/2, TLS termination
- Folosești ECS, EKS sau Lambda
- Ai nevoie de autentificare, redirecturi sau cookie-based stickiness

---

## 🔑 Functionalități cheie
<img width="1113" height="550" alt="image" src="https://github.com/user-attachments/assets/8a70aca9-b0d5-4119-abc4-66fccb618b92" />

<img width="1083" height="543" alt="image" src="https://github.com/user-attachments/assets/ae89cc40-52f4-4ef3-989c-2622c4ef6b9a" />


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

<img width="1113" height="530" alt="image" src="https://github.com/user-attachments/assets/683e9f9f-06eb-48c3-b52a-5e1df47a3e21" />


<img width="1111" height="556" alt="image" src="https://github.com/user-attachments/assets/778f81d2-bb92-4401-ba81-e99d03f47e41" />

<img width="1105" height="538" alt="image" src="https://github.com/user-attachments/assets/acc08c1b-7c30-4eda-b8a3-60abe14a255a" />

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

<img width="1059" height="533" alt="image" src="https://github.com/user-attachments/assets/2b289c72-1d35-4a39-a44a-ba4ea28eaf4e" />

Activează Sticky Sessions în target group:
```hcl
stickiness {
  type            = "lb_cookie"
  enabled         = true
  cookie_duration = 3600 # secunde
}
```
<img width="1053" height="542" alt="image" src="https://github.com/user-attachments/assets/43f87adc-fee5-4082-80f4-ffff859b143b" />


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
---

## 🖱️ Cum activezi Sticky Sessions (Stickiness) din AWS Console (GUI)

### 🔐 Scenariu: Ai un ALB și vrei să te asiguri că un utilizator este trimis către aceeași instanță backend cât timp sesiunea sa este activă.

### 🔹 Pași pentru a activa Sticky Sessions:

1. Accesează **AWS Console** → Navighează la **EC2** → secțiunea **Load Balancers**.
2. Selectează ALB-ul dorit.
3. Mergi la tabul **Target Groups** din meniul lateral stânga.
4. Selectează target group-ul asociat aplicației tale (ex: `my-app-tg`).
5. Click pe tabul **Attributes**.
6. La secțiunea **Stickiness**, apasă pe **Edit**.
7. Activează opțiunea **Stickiness enabled**.
8. Selectează tipul de stickiness:
   - `Application-based (lb_cookie)`
9. Setează durata (ex: `3600` secunde = 1 oră).
10. Apasă **Save changes**.

🎯 După acești pași, utilizatorul va primi un cookie `AWSALB=...` care îl va păstra pe același backend până expiră cookie-ul sau instanța devine indisponibilă.

---


---


---

## 🛡️ Security Group pentru ALB

<img width="1115" height="551" alt="Screenshot 2025-07-17 at 12 34 03" src="https://github.com/user-attachments/assets/81836dde-bda9-42a1-b56b-ba88a615d81e" />

### 🔄 Flux de trafic (simplificat)

```
[User / Browser] 
       │
       ▼
[ Application Load Balancer (ALB) ]
       │
       ▼
[ Target Group (EC2 / ECS / EKS / Lambda) ]
```

---

## 📥 INBOUND (User face trafic către ALB – definit în SG-ul atașat ALB)

| Protocol | Port | Source      | Scop                      |
|----------|------|-------------|---------------------------|
| TCP      | 80   | 0.0.0.0/0   | Permite trafic HTTP       |
| TCP      | 443  | 0.0.0.0/0   | Permite trafic HTTPS      |

- Aceste reguli permit acces public către aplicația ta.
- Poți restricționa `Source` la IP-uri private (ex: VPN, rețele interne).

---

## 📤 OUTBOUND (trafic de la ALB spre targeturi – controlat de target)

> ALB nu are nevoie de reguli OUTBOUND specifice.  
> **Targetul** (ex: EC2) trebuie să permită INBOUND **de la ALB Security Group**.

---

## 🎯 EC2 / ECS / EKS: Security Group Target

| Protocol | Port  | Source            | Scop                                   |
|----------|-------|-------------------|----------------------------------------|
| TCP      | 8080  | SG-ul ALB-ului    | Permite ALB să trimită trafic spre EC2 |

> Dacă aplicația rulează pe alt port (ex: 3000, 5000), setează acel port în INBOUND SG-ul instanței.

---

## 🧠 Recomandare DevOps:

- Nu folosi `0.0.0.0/0` pentru EC2 direct – folosește `source = SG-ul ALB`
- Verifică că **portul backendului** corespunde cu cel din Target Group
- În AWS Console, SG-ul ALB-ului se configurează la crearea load balancerului sau ulterior din tabul **Description**

---

## ❤️ Health Check în ALB

ALB verifică în mod periodic starea backend-urilor (EC2, ECS, IP etc.) printr-un mecanism numit **health check**.

### 🔹 Ce face?
Trimite cereri HTTP/HTTPS către un endpoint definit (ex: `/health`) pentru a vedea dacă targetul răspunde corespunzător.

### ⚙️ Parametri principali:

| Parametru              | Descriere |
|------------------------|-----------|
| **Path**               | Ex: `/health` – endpoint pe care se face verificarea |
| **Protocol**           | `HTTP` sau `HTTPS` |
| **Port**               | Portul targetului (ex: 80, 8080) |
| **Healthy threshold**  | Numărul de răspunsuri bune până când targetul devine "healthy" |
| **Unhealthy threshold**| Numărul de răspunsuri greșite până când targetul devine "unhealthy" |
| **Timeout**            | Cât timp așteaptă ALB un răspuns (ex: 5 secunde) |
| **Interval**           | Cât de des se face health check-ul (ex: la 30 secunde) |
| **Matcher**            | Codul HTTP așteptat (ex: `200`) |

### ✅ Exemplu din Terraform:

```hcl
health_check {
  path                = "/health"
  protocol            = "HTTP"
  port                = "traffic-port"
  interval            = 30
  timeout             = 5
  healthy_threshold   = 3
  unhealthy_threshold = 2
  matcher             = "200"
}
```

### 🧠 Recomandări DevOps:

- Endpoint-ul `/health` trebuie să răspundă rapid și simplu (ex: fără baze de date sau autentificare).
- Nu folosi `readinessProbe` Kubernetes complicat pentru health check extern – folosește ceva dedicat (ex: returnează `200 OK` direct).
- Health check-ul influențează dacă ALB mai trimite trafic la acel target → deci este critic pentru uptime.

---

---

## 🖱️ Cum activezi Sticky Sessions în AWS Console (fără imagini)

### 🔐 Scenariu:
Ai un Application Load Balancer configurat (ALB) și vrei ca utilizatorii să fie trimiși mereu la aceeași instanță backend pentru toată durata sesiunii lor.

---

### 🔹 Pași detaliați:

#### ✅ Pasul 1: Selectează Load Balancer-ul
1. Intră în consola [AWS Management Console](https://console.aws.amazon.com/)
2. Navighează la serviciul **EC2**
3. În meniul din stânga, mergi la **"Load Balancers"**
4. Selectează ALB-ul dorit din listă (de tip `application`)

---

#### ✅ Pasul 2: Accesează Target Groups
1. Tot în meniul lateral din EC2, accesează secțiunea **"Target Groups"**
2. Găsește Target Group-ul asociat listener-ului ALB (de obicei e legat de portul 80 sau 443)
3. Dă click pe Target Group

---

#### ✅ Pasul 3: Activează Sticky Sessions
1. După ce ai intrat în pagina target group-ului, mergi la tabul **"Attributes"**
2. Click pe butonul **"Edit"**
3. La secțiunea **"Stickiness"**, activează opțiunea:
   - `Stickiness: Enabled`
   - `Type: Application-based (lb_cookie)`
4. Setează durata în secunde (ex: `3600` pentru 1 oră)
5. Apasă **"Save changes"**

---

### 📌 Rezultat:
- ALB va trimite un cookie `AWSALB=...` către client.
- Cât timp cookie-ul este valabil, toate requesturile utilizatorului vor fi direcționate către **același target** (ex: același EC2).
- Dacă targetul devine `unhealthy`, ALB va alege altul automat.

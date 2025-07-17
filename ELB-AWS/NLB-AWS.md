

# AWS Network Load Balancer (NLB) – Ghid complet

<img width="1086" height="533" alt="image" src="https://github.com/user-attachments/assets/87dab512-7260-4e4b-8c50-11b68642366a" />


## 🔷 Ce este NLB?

**Network Load Balancer (NLB)** este un tip de Elastic Load Balancer oferit de AWS care funcționează la nivelul **Layer 4 (Transport – TCP/UDP/TLS)**. Este proiectat pentru performanță ultra-rapidă, low latency și suport pentru milioane de conexiuni simultane.

---

## 📌 Când și de ce se folosește NLB?

| Caz de utilizare                                | Motiv pentru NLB                              |
|-------------------------------------------------|------------------------------------------------|
| Aplicații în timp real (VoIP, gaming, trading)  | Latency extrem de mic și conexiuni persistente |
| TLS passthrough (end-to-end encryption)         | NLB poate transmite traficul criptat direct    |
| Ai nevoie de adrese IP statice (EIP)            | NLB suportă IP-uri statice și EIP-uri          |
| Load balancing pentru servicii non-HTTP         | Suportă TCP, UDP, TLS fără Layer 7             |
| Integrare cu servicii on-premise via IP         | Suportă targeturi IP externe                   |

---

## 🔑 Caracteristici cheie

- **Layer 4 (TCP/UDP/TLS)**
- **Ultra low latency** (millisecunde)
- **Support pentru static IPs și Elastic IPs**
- **TLS passthrough sau termination**
- **Suportă milioane de requesturi/sec**
- **Zonă de disponibilitate multiplă (multi-AZ)**
- **Health Checks pe TCP/HTTP/HTTPS**
- **Suportă trafic intern (privat) sau public**
- **Connection-based stickiness** (cu source IP)

---

## 🧠 Diferențe vs. ALB

| Caracteristică         | NLB                   | ALB                     |
|------------------------|------------------------|--------------------------|
| Nivel OSI              | Layer 4 (TCP/UDP)      | Layer 7 (HTTP/HTTPS)     |
| Protocoluri suportate  | TCP, UDP, TLS          | HTTP, HTTPS              |
| Static IP              | ✅                     | ❌                       |
| Routing pe path/host   | ❌                     | ✅                       |
| Performanță            | Ultra rapid            | Medie                    |
| WebSocket support      | ✅                     | ✅                       |
| Ideal pentru           | Aplicații rapide/non-HTTP | Aplicații web cu routing |

---

## ⚙️ Componente

- **Load Balancer (NLB)**
- **Listeners (ex: TCP 443, TCP 25)**
- **Target Group (IP, EC2, Lambda)**
- **Health Check (TCP sau HTTP/HTTPS)**

---

## 🔐 Securitate

- NLB folosește **Security Group-urile targeturilor**, NU ale load balancerului (NLB NU are SG propriu)
- Suportă **TLS passthrough** sau **TLS termination**
- Poate fi conectat la AWS Certificate Manager dacă faci termination

---

## 📥 IP-uri statice

- Poți aloca **Elastic IPs** fiecărei AZ în care rulează NLB
- Ideal pentru whitelisting firewall on-prem sau configurări DNS stricte

---

## 🧪 Health Check Example

- Protocol: TCP
- Port: 8080
- Interval: 30s
- Timeout: 10s
- Healthy threshold: 3
- Unhealthy threshold: 3

---

## 🧩 Exemple de utilizare reală

### ✅ Ex 1: Load Balancing pentru serviciu TCP (ex: PostgreSQL)
- Ai mai multe instanțe de PostgreSQL și vrei să distribui conexiunile
- NLB ascultă pe portul 5432 și transmite conexiunile către EC2

### ✅ Ex 2: TLS passthrough pentru o aplicație HTTPS
- Certificatul este gestionat în backend, iar NLB transmite traficul criptat

### ✅ Ex 3: VPN concentrator
- NLB este front-end pentru un serviciu de VPN care ascultă pe UDP

---

## 💵 Costuri

- Tarif per oră pentru NLB activ
- Tarif per LCU (Load Balancer Capacity Unit)
  - Include număr de conexiuni active, trafic și noile conexiuni/sec

---

## 🧠 Recomandări DevOps

- Folosește NLB când ai nevoie de **rapiditate, IP fix** și **non-HTTP routing**
- Nu folosi NLB dacă ai nevoie de routing L7 (folosește ALB în acel caz)
- Ideal pentru proxy-uri, baze de date, mesagerie, WebSocket low-latency

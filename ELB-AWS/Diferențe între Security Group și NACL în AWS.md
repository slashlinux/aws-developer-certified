# 🔐 Diferențe între Security Group și NACL în AWS

## 🧾 Tabel comparativ

| Caracteristică                        | Security Group (SG)            | Network ACL (NACL)             |
|--------------------------------------|--------------------------------|--------------------------------|
| **Tip**                              | Firewall la nivel de instanță  | Firewall la nivel de subnet    |
| **Stateful**                         | ✅ Da – **ține minte** conexiunile | ❌ Nu – **nu ține minte**, trebuie specificat inbound/outbound |
| **Scop**                             | Protejează instanțele EC2, ELB etc. | Protejează întregul subnet     |
| **Direcții de trafic**               | Doar **inbound** e configurabil, outbound e permis implicit (dar poate fi modificat) | Trebuie configurate **inbound și outbound** separat |
| **Implicit**                         | Totul e blocat (deny all), apoi adaugi reguli | Totul e permis (allow all), apoi restricționezi |
| **Aplicație**                        | Se atașează la **EC2, ENI, ELB** | Se atașează la **subneturi**   |
| **Rang de porturi**                  | ✅ Suportă porturi individuale și range-uri | ✅ Da, porturi și range-uri    |
| **Reguli per direcție**              | Până la ~60 reguli              | 20 reguli implicite (poate fi extins) |
| **Logging**                          | ❌ Nu suportă loguri native     | ✅ Suportă **flow logs**        |
| **Ordinea regulilor**                | ❌ Nu contează ordinea          | ✅ Ordinea contează (prima regulă care se potrivește se aplică) |

---

## 🔍 Exemple:

### 🔸 Security Group (SG)
- Permite **TCP port 22** din `203.0.113.10/32` (SSH doar dintr-un IP)
- Traficul de răspuns (outbound) e automat permis pentru acea sesiune

### 🔸 NACL
- Dacă permiți **TCP port 80 inbound**, trebuie să adaugi și o **regulă outbound pentru răspuns**
- Dacă nu adaugi, răspunsul va fi blocat (pentru că NACL este **stateless**)

---

## 🧠 Recomandări DevOps

- Folosește **Security Groups** pentru control fin pe instanțe și servicii (EC2, ALB, RDS)
- Folosește **NACLs** pentru politici largi de rețea (ex: blocare IP-uri la nivel de subnet)
- Pentru maximum de control: ✅ combină **SG + NACL**
- Activează **VPC Flow Logs** pentru troubleshooting rețea

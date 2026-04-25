# 🌐 DNS (Domain Name System)

## 📌 1. What is DNS?
DNS converts a domain name (like `linkedin.com`) into an IP address.

👉 It works like an internet phonebook.

---

## 🧠 2. Key Components

### 🌍 Root Servers
- Top level of DNS hierarchy
- Managed under ICANN
- 13 logical servers (A–M)
- Do NOT return IP

👉 They say:
"Ask the TLD server (.com, .org)"

---

### 🌐 TLD Servers (.com, .in, etc.)
- Handle domain extensions
- `.com` is managed by VeriSign

👉 They say:
"Ask the authoritative name server"

---

### 🏁 Authoritative Name Server
- Stores actual DNS records
- Returns final IP

Example:
linkedin.com → 150.171.22.12

---

### 🔍 DNS Resolver
- Example: ISP DNS (Jio, Google DNS)
- Performs full lookup
- Caches results

---

## 🔄 3. DNS Resolution Flow

Browser → OS → DNS Resolver → Root → TLD → Authoritative → IP

Step-by-step:
1. Browser asks OS
2. OS asks DNS resolver
3. Resolver queries root server
4. Root → points to TLD (.com)
5. TLD → points to authoritative server
6. Authoritative → returns IP
7. IP sent back to browser

---

## 🔁 4. Recursive vs Iterative

### ✅ Recursive
Client asks resolver:
"Give me final IP"

Resolver does everything.

---

### 🔁 Iterative
Client follows steps manually:

Client → Root → TLD → NS → IP

---

## 🌐 5. Real-World Observation

### ❗ IP is NOT actual backend server

Example:
linkedin.com → 150.171.22.12

👉 This is:
- Edge / proxy server
- Not actual backend

---

## 🏗️ 6. Real Architecture

User  
 ↓  
DNS → Edge IP  
 ↓  
Load Balancer / CDN  
 ↓  
Backend Servers  

---

## ⚠️ 7. Why direct IP failed

Error:
"Our services aren't available right now"

👉 Reason:
- Virtual hosting
- Server needs domain (Host header)

---

## 🧠 8. Key Learnings

- DNS is hierarchical
- Root doesn’t know IP
- TLD doesn’t know IP
- Only authoritative server knows IP
- Resolver handles complexity
- IP = entry point (not backend)

---

## 🚀 9. Quick Revision

Domain → Resolver → Root → TLD → NS → IP  
IP → Edge → Load Balancer → Backend  

---

## ✍️ 10. Personal Notes

(Add your own insights here)

- DNS hides complexity using recursion
- IP is not actual server
- CDN + proxy used in real systems

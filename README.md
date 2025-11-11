# Tugas-1-Jarkom_M-Faqih-Ridho_123


# TUGAS SUBNETTING & ROUTING — 5027241123

## 1) Ringkasan
- **NRP**: 5027241123 → basis IP **10.163.0.0** (1123 mod 256 = 163).
- **Topologi**: 2 router — **R‑HQ** (pusat) dan **R‑Branch** (cabang).
  - R‑HQ **Gi0/0** (trunk, subinterface VLAN 100/200/300/400/500) → Switch Pusat.
  - R‑HQ **Gi0/1** ↔ **R‑Branch Gi0/2** (link **/30**).
  - R‑Branch **Gi0/0** → LAN Pengawas.
- **Routing**: *Static route*.
- **Alamat supernet seluruh jaringan**: **10.163.0.0/22**.

---

## 2) Tabel VLSM
| Nama | Network | Mask | Prefix | Range Host | Broadcast | Gateway |
|---|---|---|:---:|---|---|---|
| Sekretariat | 10.163.0.0 | 255.255.254.0 | /23 | 10.163.0.1 – 10.163.1.254 | 10.163.1.255 | 10.163.0.1 |
| Bidang Kurikulum | 10.163.2.0 | 255.255.255.0 | /24 | 10.163.2.1 – 10.163.2.254 | 10.163.2.255 | 10.163.2.1 |
| Bidang Guru & Tendik | 10.163.3.0 | 255.255.255.128 | /25 | 10.163.3.1 – 10.163.3.126 | 10.163.3.127 | 10.163.3.1 |
| Bidang Sarana Prasarana | 10.163.3.128 | 255.255.255.192 | /26 | 10.163.3.129 – 10.163.3.190 | 10.163.3.191 | 10.163.3.129 |
| Bidang Pengawas (Cabang) | 10.163.3.192 | 255.255.255.224 | /27 | 10.163.3.193 – 10.163.3.222 | 10.163.3.223 | 10.163.3.193 |
| Server & Admin | 10.163.3.224 | 255.255.255.248 | /29 | 10.163.3.225 – 10.163.3.230 | 10.163.3.231 | 10.163.3.225 |
| Link R‑HQ ↔ R‑Branch | 10.163.3.232 | 255.255.255.252 | /30 | 10.163.3.233 – 10.163.3.234 | 10.163.3.235 | R‑HQ: 10.163.3.233 / R‑Branch: 10.163.3.234 |

> Catatan: IP tepat di batas (mis. **10.163.3.128**) adalah **network address** untuk subnet berikutnya, sehingga **bukan** IP host.

---

## 3) Ringkasan CIDR & Supernetting
- Gabungkan subnet kecil 10.163.3.x → **10.163.3.0/24**.
- 10.163.2.0/24 + 10.163.3.0/24 → **10.163.2.0/23**.
- 10.163.0.0/23 + 10.163.2.0/23 → **10.163.0.0/22** .
- Wildcard mask supernet: **0.0.3.255**.

---

## 4) Konfigurasi Inti

### 4.3. Switch Pusat
```console
conf t
vlan 100 name SEKRETARIAT
vlan 200 name KURIKULUM
vlan 300 name GURU_TENDIK
vlan 400 name SARPRAS
vlan 500 name SERVER_ADMIN

interface fa0/1
 description trunk_to_R-HQ_gi0/0
 switchport mode trunk
 switchport trunk allowed vlan 100,200,300,400,500
 spanning-tree portfast trunk
end
write memory
```



### 4.2. R‑Cabang
```console
conf t
hostname R-Branch
no ip domain-lookup
no router rip

interface gi0/0
 description PENGAWAS
 ip address 10.163.3.193 255.255.255.224
 no shutdown
 exit

interface gi0/2
 description Link_to_R-HQ
 ip address 10.163.3.234 255.255.255.252
 no shutdown
 exit

! Ringkas semua jaringan pusat
ip route 10.163.0.0 255.255.252.0 10.163.3.233
end
write memory
```
---

## 5) Verifikasi Cepat
1. **Link /30**: `R-HQ# ping 10.163.3.234` dan `R-Branch# ping 10.163.3.233` → reply.
2. **Subinterface up**: `R-HQ# show ip interface brief` (gi0/0.x up/up).
3. **Trunk benar**: `Switch# show interfaces trunk` (Fa0/1 allow 100,200,300,400,500).
4. **PC**: IP & gateway sesuai tabel; ping antar-VLAN & ke Pengawas.

---

## 6) Troubleshooting Singkat
- PT crash saat paste → tempel **per blok** (5–8 baris) dan simpan tiap selesai.
- Subinterface down/down → pastikan **Gi0/0 tidak punya IP** & port switch **mode trunk**.
- Ping antar-site gagal → cek **static route** di kedua router dan IP link **/30**.


**Tabel hasil VSLM**

<img width="746" height="85" alt="image" src="https://github.com/user-attachments/assets/a09240c6-6624-42d4-9a1e-eea07492fc45" />

**Tabel hasil CIDR**

<img width="757" height="62" alt="image" src="https://github.com/user-attachments/assets/45d5e23d-4ef7-43f4-8156-d4eba618b0c9" />

**Supernetting Visual**
```
10.163.0.0/22

├─ 10.163.0.0/23(Sekretariat)

└─ 10.163.2.0/23

   ├─ 10.163.2.0/24     (Kurikulum)
   
   └─ 10.163.3.0/24
   
      ├─ 10.163.3.0/25      (Guru & Tendik)
      
      ├─ 10.163.3.128/26    (Sarpras)
      
      ├─ 10.163.3.192/27    (Pengawas - Cabang)
      
      ├─ 10.163.3.224/29    (Server & Admin)
      
      └─ 10.163.3.232/30    (Link R-HQ ↔ R-Branch)
```


**Topologi**

<img width="1089" height="595" alt="image" src="https://github.com/user-attachments/assets/fe1246b0-873a-4e8d-bbf8-9825db35d598" />



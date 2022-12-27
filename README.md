# DNS


Ya sabes convierte los nombres en direcciones IPs para  que sean alcanzables 

## Estructura ( pero mas bien es la respresentacion grafica de como se guardan los nombres de dominios en internet y de aqui se deriva como se buscaria un dominio)

Se dice que es como un arbol proponen dos ejemplos primero proponen el fully qualified domain name (FQDN):

```
www.domain-A.com
```

entonces nos dan una imagen para ilustrar como parece un arbol la estructura.


![image](https://user-images.githubusercontent.com/63270579/209502336-9be29db5-169f-4ca1-81d2-23c6588e864b.png)


El segundo ejemplo en "inlanefreight.com"


![image](https://user-images.githubusercontent.com/63270579/209502608-20ed5bb2-afa8-40d8-a5b4-c64ca0ab17c5.png)

Nos dice que 

>Each domain consists of at least two parts:

1. Top-Level Domain (TLD) (el .com)


2. Domain Name ( inlanefreight )

> From the last example, the domain name would be "inlanefreight" and the TLD then "com". If we look at the "inlanefreight," we see that it contains some so-called subdomains (dev, www, mail). These subdomains can represent a single host or virtual hosts (vHosts). ( no sabia que el www era un subdominio )


## The components of a DNS

### Name servers ( servidores de nombres )

Los servidores de nombres contienen las denominadas zonas o archivos de zona. En términos simples, los archivos de zona son listas de entradas para el dominio correspondiente. Podemos pensar en ello como una guía telefónica para una ciudad específica. Estos archivos de zona contienen direcciones IP para dominios y hosts específicos. Dicho archivo de zona existe en todos los sistemas, ya sea Linux o Windows, y se denomina archivo "hosts". Podemos encontrarlos en Pwnbox en "/etc/hosts". Este archivo es nuestro archivo de zona local.

### Zones

Aqui lo explica muy bien en el link pero la zonas como para separar.  Las zonas DNS no están necesariamente separadas físicamente entre sí, se usan estrictamente para delegar el control. con esta cita seguro te vuelve la idea 



https://www.cloudflare.com/es-es/learning/dns/glossary/dns-zone/

### Domain names

Los nombres del dominio 

### IP addresses

Las direcciones ips ve el archivo /etc/host  ( asi me imagino que estan dentro de los archivos de zona al parecer asi es)

## The DNS servers ( entiendo que son con los que te conectas cuando resuelves un nombre)

> Are divided into four different types: 

Para resolver un nombre se ultilizan varios Name servers no solo uno.  el proceso es el siguiente segun entiendo 



### Recursive resolvers (DNS Recursor)

Es como un intermediario que se encarga de guardar en la cache las peticiones pero es un intermediario entre el cliente y el name server. algo asi como cloadflare o una CDN. ( solo es un ejemplo mio )

### Root name server

Se puede acceder a trece servidores de nombres raíz con direcciones IPv4 e IPv6. Estos contienen los archivos de zona de  todos los nombres de dominio y direcciones IP de los TLD. 

>  Every recursive resolver knows these 13 root name servers.

Tienen estos 13 servidores una letra asignada de la `"a" a la "m". Se encuntran en el dominio "root-servers.net"

```
dig ns root-servers.net | grep NS | sort -u
```

Estos 13 servidores de nombres raíz representan los 13 tipos diferentes de servidores de nombres raíz. No significa que solo se distribuya en 13 hosts, sino en más de 600 copias de estos servidores de nombres raíz en todo el mundo.

Por lo que puedo entender cuanod se hace una query es la primera parada este servidor acepta la peticion que viene del recursive reolver y redirecciona y responde a la peticion al correspondiente TLD server.

> Each of these root name servers accepts a recursive resolver query that contains a domain name. The domains' extension answers the query and forwards the recursive resolver to the corresponding TLD name server. 



### TLD name server

"A TLD name server manages the information on all domain names that have the same TLD" ( por ejemplo .com) Eso quiere decir que "This means that all domains under the TLD ".com" are managed by the corresponding TLD name server" 

EL flujo de datos seria el siguiente: "When we look for a domain registered under this TLD, the recursive resolver, after receiving a response from a root name server, sends a query to a TLD name server responsible for that TLD."

### Authoritative name servers

Este es el ultimo paso el TDL Redirecciona aqui y tiene la ip otra informacion como las DNS entries:

> Authoritative name servers store DNS record information for domains. These servers are responsible for providing answers to requests from name servers with the IP address and other DNS entries for a web page so the web page can be addressed and accessed by the client. When a recursive resolver receives a TLD name server response, the response refers it to an authoritative name server. The authoritative name server is the last step to get an IP address.


EN resumen lo que entendi fue: El cliente hace una peticion para resolver un nombre este va al ***"Recursive resolvers"*** estos primero checan su cache en busca de peticiones similares si no hay el segundo paso es enviar una peticion al uno de los ***"root server"*** atiende la peticion dirigiendo al recursive resolver al
***"TDL"*** dependiendo si es .com .net o www entre muchos otros. Finalmente ***"Authoritative Name Server"*** tiene toda la informacion del dominio la ip y lo demas registros.


## DNS Zones

### Primary DNS Server

Es el servidor que tiene la informacion de los zone files lo que entiendo es que este es donde se modifica el archivo zone y se replica a los servidores secundarios esto con el objetivo de tener un backup. si no esta este disponible pues e consulta al secundario.

### Secondary DNS Server

> Los servidores DNS secundarios contienen copias de solo lectura del archivo de zona del servidor DNS principal. Estos servidores comparan sus datos con el servidor DNS principal a intervalos regulares y, por lo tanto, sirven como servidor de respaldo. Es útil porque la falla de un servidor de nombres principal significa que las conexiones sin resolución de nombres ya no son posibles. De todos modos, para establecer conexiones, el usuario tendría que conocer las direcciones IP de los servidores contactados. Sin embargo, esta no es la regla.

### Zone Transfer

Se le llama trasferencia de zona cuando se copia informacion del DNS primary a uno secundario ( puede haber muchos de estos )
y existen 2 metodos que en general se diferencian de si se copia toda la informacion(AXFR - Asynchronous Full Transfer Zone) o bien si solo se copia las entradas que han sido agregadas o modfiicadas (IXFR - Incremental Zone Transfer).


### Ataque Zone Trasfer

> The problem with DNS servers and zone transfers is that it does not require authentication and can be requested by any client. If the administrator has not set "Trusted Hosts/IP addresses" for the DNS servers, which have permission to receive these zones, we can also query the entire zone file with its contents. This will give us all IP addresses with the respective hosts and significantly increase our attack vector if the agreement does not limit our scope. Even today, it is quite common that the DNS servers are misconfigured and allow this.


### DNS Reconocimiento

Ya habiendo estudiado como funciona el protocolo DNS podemos recopilar informacion que informacion es util:

1. DNS Records (MX, TXT, A)

2. Subdomains/Hosts

3. DNS Security


## OSINT

Para sacar datos se usan herramientas de fuentes publicas algunas son:

```
 Google Dorks
 VirusTotal
 DNSdumpster
 Netcraft
 
```

## Certificate Transparency

Es una herramienta donde al parecer se registran certificados que usan las paginas para el protocolo TLS y es informacion publica que puede ser usada para ver que paginas tiene cierta organizacion.

> Certificate Transparency (CT) logs contain all certificates issued by a participating Certificate Authority (CA) for a specific domain. Therefore, SSL/TLS certificates from web servers include domain names, subdomain names, and email addresses. Since these logs are public and accessible to everyone, it is a valuable source for understanding the target company's infrastructure better. We can use a tool that outputs all the CT logs for our target domain from different sources and filtered is ctfr.py.

> Los registros de transparencia de certificados (CT) contienen todos los certificados emitidos por una autoridad de certificación (CA) participante para un dominio específico. Por lo tanto, los certificados SSL/TLS de los servidores web incluyen nombres de dominio, nombres de subdominio y direcciones de correo electrónico.

### CTFR.py

```

python3 ctfr.py -d inlanefreight.com

```


## Zone Transfer

Ya sabes trasferir las zonas de un servidor a otro ;) 

### Assesment

Cuantos name servers 

```
dig axfr  @10.129.105.63 inlanefreight.htb
Command : dig axfr cronos.htb @10.10.10.13 # la ip de la maquina tenia el puerto 53 abierto y ese es el dominio que tambien estaba o era la maquina Cronos ya entendi con esto como se usa.
```


## Python PIP8


Es una guia para codificar codigo de calidad 

> Hay pautas desarrolladas especialmente para Python, conocidas como Python Enhancement Proposal( PEP8). PEP8 es un documento que contiene guidelinesy best practicespara escribir código Python y fue escrito en 2001 por Guido van Rossum, Barry Varsovia y Nick Coghlan. PEP8El enfoque principal de es mejorar el readabilityy consistencydel código de Python. Cuando tengamos más experiencia escribiendo código Python, podremos empezar a trabajar con otros. Escribir código legible es crucial porque otras personas que no están familiarizadas con nosotros coding stylenecesitan leer y comprender nuestro código. Si tenemos pautas que seguimos y reconocemos, otros también encontrarán nuestro código más fácil de leer.




























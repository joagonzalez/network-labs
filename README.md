Repositorio de laboratorios

### Laboratorio l3vpn

![Figura 1](https://raw.githubusercontent.com/joagonzalez/networkLabs/master/l3vpn/mpls-te.png)


### Descripción

- Se define un core MPLS con múltiples caminos en la topología física con el objetivo de probar configuraciones de TE con LDP, RSVP tanto para intServ como diffServ

- Por defecto, el core MPLS estará configurado con LDP para dar servicios de l3vpn. Se utilizará OSPF como IGP dentro de la red de transporte

- LDP utilizará las rutas seleccionadas por el IGP (en este caso con algoritmo SPF) para seleccionar el mejor LSP dentro de la LFIB (Lable Forwarding Information Base). Ver ejemplo práctico en R3, donde se selecciona R4 como próximo salto para alcanzar la lo0 de R5 (5.5.5.5/32)

- Se configuran VRFs por cliente en los PE y se establecen sesiones BGP entre cada sitio y las fronteras de la red de transporte con el objetivo de anuncias las rutas de cada sitio

- Las rutas son redistribuidas internamente a través de comunidades BGP utilizando Route Distinguishers y Router Targets (Multi protocol BGP VPN v4)

- TE podrá configurarse con RSVP o BGP

LIB router 3:
```
  lib entry: 5.5.5.5/32, rev 12
	local binding:  label: 301
	remote binding: lsr: 4.4.4.4:0, label: 401
	remote binding: lsr: 7.7.7.7:0, label: 600
	remote binding: lsr: 2.2.2.2:0, label: 201
```

Si bien alcanza el prefijo 5.5.5.5/32 por R4 y R7, en la FLIB se instala solamente el LSP que pasa por R4 ya que es la ruta mas corta seleccionada por OSPF.

```
R3#sh mpls forwarding-table 
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
300        No Label   7.7.7.7/32       0             Fa1/1.36   10.0.36.2   
301        401        5.5.5.5/32       0             Fa1/0.34   10.0.34.4   
302        Pop Label  4.4.4.4/32       0             Fa1/0.34   10.0.34.4   
303        Pop Label  2.2.2.2/32       0             Fa0/0.23   10.0.23.2   
304        Pop Label  10.0.63.0/24     0             Fa1/0.34   10.0.34.4   
           Pop Label  10.0.63.0/24     0             Fa1/1.36   10.0.36.2   
305        Pop Label  10.0.45.0/24     0             Fa1/0.34   10.0.34.4  
```

Tabla de enrutamiento R3:

```
      5.0.0.0/32 is subnetted, 1 subnets
O        5.5.5.5 [110/3] via 10.0.34.4, 00:19:30, FastEthernet1/0.34

```
# Born2beroot

Estudiando comandos por ssh
ssh dgomez-m@localhost -p 4242 para conectarnos desde cualquier terminal a nuestra virtual machine 
Desde Powershell o Macos conseguimos conectarnos.

Estudiando el bash:
## Obtener la arquitectura 
  - comando uname te devuelve el sistema
  - uname -a (--all) te devuelve sistema arquitectura tipo de so
## Nucleos fisicos
  - Aqui utilizaremos el comando grep para buscar dentro del archivo /proc/cpuinfo por "physical id" Esto nos devolvera la linea de physical id
  - El problema es que habra tantas lineas iguales por numero de de nuclos por ejemplo en mi maquina virtual solo me devuelve una linea physical id = 0 en mi pc de casa 28 lineas physical id entonces necesitamos una forma de contarlas entonces usaremos el comando tuberia para que nos cuente el numero de lineas 
## Nucleos virtuales
  Lo mismo pero en este caso contaremos las lineas proccesor
  grep processor /proc/cpuinfo | wc -l
## Memoria ram
  - El comando free nos muestra informacion sobre la ram
  - el total , usado , la memoria libre, etc
  - para la informacion en megas necesitamos --mega ya que -m son mebibytes y a nadie le importa ese formato
  - El comando Awk
  - Es una herramienta potente para procesar textos , en el contexto que estamos usando es por que queremos sacar cierta informacion de free --mega
si uamos el operador tuberia despues del free
podemos acceder a cada linea y recuperar cosas 
```bash
dgomez-m@dgomez-m42:~$ free
               total        used        free      shared  buff/cache   available
Mem:         2014520      221808     1823196         596       90000     1792712
Swap:        2244604           0     2244604
```
free -- mega | awk '$s1 == "Mem:"'
Busca  la linea donde Ponga Mem 
```bash
free --mega | awk '$1 == "Mem:"'
Mem:            2062         227        1866           0          92        1835
```
y si queremos recuperar un valor en concreto usamos 
free --mega | awk '$1 == "Mem:" {print $Numerodepalabra}'
Donde numero de palabra sera el valor que querramos recuperar por ejemplp el total seria 
```bash
dgomez-m@dgomez-m42:~$ free --mega | awk '$1 == "Mem:" {print $2}'
2062
```
Para la parte del porcentaje usaremos printf 
veamos este comando 
## calculo de porcentaje de memoria usada
```bash
dgomez-m@dgomez-m42:~$ free --mega | awk '$1 == "Mem:" {printf("(%d)\n", $3/$2*100)}'
(11)
```
Este printf nos muestra en entero el porcentaje de memoria usada para hacer el calculo usamos la memoria usada entre el total por cien

Ahora bien si queremos hacer lo que nos piden
con dos decimales y % al final tenemos que trabajarlo un poquito mas
```bash
free --mega | awk '$1 == "Mem:" {printf("(%f)\n", $3/$2*100)}'
(11.008729)
```
con  %f de float nos imprime todos los decimales pero como solo queremos 2 solo haria falta modificarlo %.2f
 y como queremos el porcentaje delante y ya lo tenemos controlado del printf el comando final seria asi
 ```bash
 free --mega | awk '$1 == "Mem:" {printf("(%.2f%%)\n", $3/$2*100)}'
(10.96%)
```







    

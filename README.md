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
## Memoria del disco

Para ver el estado de memoria del disco usamos el comando df


![image](https://github.com/nakamavg/Born2beroot/assets/7202262/ecf40f75-c37b-4f7f-a6b1-e956fd80d193)

Para que nos muestre solo las lineas que queremos /dev/ y nos excluya las lineas que contengan /boot
usamos grep para filtrar lo que queremos y grep -v para que no nos incluya esas lineas 

`df -m | grep "/dev/" | grep -v "/boot"`

![image](https://github.com/nakamavg/Born2beroot/assets/7202262/7b6ec911-cf44-49cf-b08e-494fdace8c30)

ademas vamos a usar el comando awk para quie nos sume todo el espacio usado por estas carpetas

`df -m | grep "/dev/" | grep -v "/boot" | awk '{memory_use += $3} END {print memory_use}'`
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/81321982-d7a0-4e68-a49c-c88a8aade9a6)

Una vez obtenido el espacio usado necesitamos el espacio total , que es un comando muy parecido
`df -m | grep "/dev/" | grep -v "/boot" | awk '{memory_result += $2} END {printf ("%.0fGb\n"), memory_result/1024}'`

aqui le estamos diciendo que nos sume los valores en s2 y que nos haga un printf de los valores sumados dividido entre 1024 para obtenerlo en GB
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/f30ff72c-5264-4f2e-93e0-0592011aafc8)

Tambien debe salir el porcentaje de la memoria usada
para ello sumaremos todos los valores de memororia usada y los guradaremos en use sumaremos toda la memoria total y la guardaremos en total al final imprimiremos el use entre el total multiplicado por 100 para obtener el porcentaje.
usando en el printf %d para quitarnos los decimales .
`df -m | grep "/dev/" | grep -v "/boot" | awk '{use += $3} {total += $2} END {printf("(%d%%)\n"), use/total*100}'`
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/39a86858-1489-4403-9630-4b7387153e9e)

## Porcentaje uso de CPU
el comando vmstat
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/05eee097-7291-4572-b3b1-7ac3a447271a)

### Memory informacion sobre el uso de la memoria
- R : nos muestra el numero de procesos en ejecucion
- b : procesos en espera
- swpd : la cantidad de memoria virtual utilizada
- free : la cantiddad de memoria fisica disponible
- buff : la cantidad de memoria utilizada como memoria intermedia del sistema
- cache : la memoria cache utilizada por el sistema
### swap espacio de intercambio
- si : la cantidad de memoria intercambiada desde el disco al espacio de swap por segundo
- so : la cantidad de memoria intercambiada desde swap al disco por segundo

### io Entrada y salida
- bi: la cantidad de bloques leidos desde dispositivos por segundo
- bo: la cantidad de bloques escritos en dispositivos por segundo

### system información sobre la actividad del sistema

- in : cantidad de interrupciones por segundo
- cs : cantidad de cambios de contexto por segundo

### cpu

- us : porcentaje de tiempo utilizado en procesos de usuario
- sy : porcentaje de tiempo utilizado en procesos del sistema
- id: porcentaje de tiempo de cpu inactiva
- wa: porcentaje de tiempo esperando entrada y salida
- st : indica el procentaje de tiempo de cpu
  robado por maquinas virtuales.

  -- Un valor alto en la columna "st" podría indicar que las máquinas virtuales en tu sistema no están obteniendo suficiente tiempo de CPU y están siendo limitadas por el hipervisor. En entornos de virtualización, es importante monitorear esta métrica para asegurarse de que las máquinas virtuales obtengan suficientes recursos para realizar sus tareas de manera eficiente.

Ahora bien para lo que nos pide el subject que es el % de cpu 
primero necesitamos solo obtener la ultima linea
para esto usaremos un comando tuberia
` vmstat | tail -1`
como solo queremos el valor 15 hacemos por tuberia 
` vmstat | tail -1 | awk '{print $15}'`
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/d764fe91-e90d-4255-86bf-9f35282b68f0)

### ultimo reinicio 

comando `who -b`

nos muestra mas de lo que queremos como solo queremos la fecha tenemos que hacer un awk

`who -b | awk '$1 == "system" {print $3 " " $4}'`
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/cba8af26-adad-4670-8dc4-20a0b29bd23d)

### uso lvm
En el subject nos piden que pongamos yes or no dependiendo de si lvm esta activo
para ello vamos a seguir usando los comandos aprendidos hasta ahora.

comando lsblk

![image](https://github.com/nakamavg/Born2beroot/assets/7202262/8cb95a54-d93b-4542-9cac-4ffb2f23608e)
Nos muestra una lista jerarquica de discos duros particiones tamaño y puntos de montaje

como queremos saber si hay lvm

lsblk | grep "lvm" | wc -l esto nos vas a contar el numero de lineas al buscar con grep lvm 
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/d2930c88-2e62-4982-96e3-06d22cc4ec8a)

ahora necesitamos que nos ponga yes or no dependiendo de si encuentra lineas con el grep

 `if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi`

 Metemos dentro del if entre corchetes el comando anterior y le metemos `- gt 0` para validar que haya mas lineas que 0
 dependiendo de el resultado de la validacion nos hace un echo , y para terminar el if debemos poner fi
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/787c5b22-4ff0-45cf-801e-cd528fcee716)

### Conexiones TCP

Para mirar el numero de conexiones TCP establecidadas usaremos:

`ss -ta | grep ESTAB | wc -l `

Esto nos contara el numero de lineas ESTAB
lo que nos devolvera el numero de conexiones
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/549b02ba-7cc5-4009-8ac5-7d9b76fec70b)

### Numero de usuarios
users | wc -w

### Dirección IP y mac 

comandos
hostname -I (devuelve ip)
ip link( devuelve mucho texto y la mac addres)
Para dejarlo bonito debemos hacer un grep de "link/ether" pasarselo por pipe a awk e imprimir solamente la segunda palabra 
![image](https://github.com/nakamavg/Born2beroot/assets/7202262/24bee760-85df-450a-a055-190f085d7b5b)

### numero de comandos ejecutados con sudo

journalctl _COMM=sudo | grep COMMAND | wc -l)










    

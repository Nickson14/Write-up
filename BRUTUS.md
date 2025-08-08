En este sencillo Sherlock, te familiarizarás con los registros auth.log y wtmp de Unix. Exploraremos un escenario en el que un servidor Confluence fue atacado por fuerza bruta a través de su servicio SSH. Tras obtener acceso al servidor, el atacante realizó actividades adicionales, que podemos rastrear mediante auth.log. Aunque auth.log se utiliza principalmente para el análisis de fuerza bruta, profundizaremos en todo el potencial de este artefacto en nuestra investigación, incluyendo aspectos de escalada de privilegios, persistencia e incluso cierta visibilidad de la ejecución de comandos.


El dia de hoy nos encontramos con la resolucion de un laboratorio sherlock de la plataforma de hackthebox en la cual mostrare una serie de pasos y explicare el por que se tiene que solucionar de esa manera. 

Para la resolucion del mismo nos dan 3 archivos uno de auth.log 

![[Pasted image 20250807122636.png]]

Para encotrar la primera pregunta necesistamos abrir el archivo de auth.log una vez con el mismo tenemos que buscar entre diferentes log cual es la ip que esta haciendo el ataque de fuerza bruta.

Podemos hacer un: 
cat auth.log | grep "from" 

Esto es un filtrado basico pero eficiente ya que nos mostrara la comunicacion y de donde se esta comunicando en caso de ver un ip externa y que cumpla con las caracteristica en este caso de un ataqie de fuerza bruta  que esto podemos revisarlo de la siguiente manera.

Podemos hacer un: 
cat auth.log | grep "Failed"
cat auth.log | greo "Accepted"

Ya que aqui vamos a ver todos los intentos de inicio de seccion o tambien filtrando por el protocolo en este caso es ssh.

cat auth.log | grep "ssh2"

Que es el cual se encuentra activo en el documento.



![[Pasted image 20250807123311.png]]

Siguiendo la logica de filtrado en caso de que el usuario hay adversario haya obtenido el acceso lo unico que necesitamos verificar es que en caso de exista un log con la palabra Accepted.

cat auth.log | greo "Accepted" 
Este puede ser una de los comandos anterios para linux.
Algo que hay que tomar en cuenta en caso de que nosotros querramos solucionarlo desde windows yo lo hago leyendo desde el notepad y solo con darle a control + f  nos abrira el apartado de filtrar por palabras y ahora veremos.

![[Pasted image 20250807123606.png]]

solo con esta palabra podemos ver quien se logio y quien no.



![[Pasted image 20250807124432.png]]
Para esto el archivo de auth.log no nos va permitir ver la hora exacta y para esto necesitamos ver el otro archivo  para esto  necesitamos el ejecutable utmp.py para descomprimir el archivo wtmp y nada.

en mi caso lo hice desde la terminal de vscode pero en linux no es mas que,

chmod +x -f /path/  "del ejecutable",  y para su funcionamiento python3 utmp.py y de parametro el archivo wtmp 
el comando se veria asi:

python3 utmp.py ./wtmp 

![[Pasted image 20250807124412.png]]


hora de correlacionar tiempo

En este caso nos piden la fecha exacta 
![[Pasted image 20250807125726.png]]
y cuando vamos al otro arhchivo nos damos cuenta que no tenemos la mima hora  ahora como podemos determinar el horario muy facil bueno hay que tener imaginacion y tener en cuenta que tal vez se ecuentren en distintos horarios lo que nos da una pista en hackthebox en el apartado de hint.

Mirando los logs del otro archivo nos damos cuenta que el log de oro de nosotros para esta pregunta es esta.
![[Pasted image 20250807125933.png]]

Ya que el mismo cuenta con la misma ip del ataque de fuerza bruta pero  al ver las horas no se relacionan entre si.

asi que lo que podemos hacer es colocar el formato pedido  intentar con la misma pero al ver que esta atrazada podemos determinar que necesitamos cambiar los digitos del 02 al 06  y nada mas pensando que en los primeros logs los cuales estan correcto esta fue la hora en la que se consiguio el acceso

![[Pasted image 20250807125652.png]]



![[Pasted image 20250808091613.png]]

Para responder esta pregunta necesitamos ver los logs de nuevo para identificar cual fue la session que se habrio una vez el atacante logro autenticarse.

![[Pasted image 20250808091956.png]]

Verificando los logs nos damos cuenta que no teniamos que bajar tanto ya que nos encontrabamos en el log necesario para identifcarla en este caso se abrio la session 34 para esta conexion.


![[Pasted image 20250808092031.png]]

***TASK 5***

![[Pasted image 20250808092112.png]]

Para esta podemos llegar desde varias maneras pero si se fijan cuando vimos los otros logs ya aparecia una conexion rara y un usuario nuevo en el sistema para esto podemos filtrar por los directorios necesarios para crear  persistencia, o por los grupos de sudo, o por comados useradd y groupad ya que son necesarios para la creacion de un usuario nuevo.

Y nada yo filtrare por el comando utilizado para creacion un usuario.

![[Pasted image 20250808092417.png]]

Ya aqui tenemos los logs relacionados y tambien podiamos filtrar por new user entre otras aqui dejare el comando en caso de que sea linux.

cat auth.log | grep "useradd"

cat auth.log | grep "new user"

Asi sucesivamente.

![[Pasted image 20250808092612.png]]


***Task 6

![[Pasted image 20250808092700.png]]

Para responder esta pregunta nos dirigimos para el apartado de matrices de ataques  de mitre.
https://attack.mitre.org/matrices/enterprise/ una vez aqui nos fijamos en la parte de persitencia ya que ahora estamos siguiendo al atacante en esta fase y necesitamos una respuesta.

![[Pasted image 20250808092940.png]]

Ahora para identificar la subtecnica tenemos que ir al apartado que nos toca en este caso nos damos cuenta que dentro del aparatado de persistencia nos dice "local account" creacion de cuenta.

![[Pasted image 20250808093213.png]]

Luego de esto nos damos cuenta que esta tactica tiene 3 subtecnicas, para resolverla es facil ya que el atacante creo la cuenta a nivel local para seguir teniendo acceso.
![[Pasted image 20250808093443.png]]


![[Pasted image 20250808093522.png]]

***Task 7 


![[Pasted image 20250808094358.png]]


Para responder esta pregunta nos vamos a los logs y sin filtrar podemos ver cuando se cerro la session, Esta tecnica no es bueno accerla cuando hay una cantidad elevada de logs pero en este caso el archivo auth.log no es muy extenso asi que podemos solo bajar un poco y darnos cuenta de cuando se cerro la session.

Recordando que el dia fue el mismo ya que la explotacion de este ataque se hizo en lapso de minutos.

![[Pasted image 20250808094631.png]]


![[Pasted image 20250808094657.png]]


Aparte que nos indicara que este es la session del atacante? si no se recuerdan en la parte de arriba nosotros mismos.


***Task 8 


![[Pasted image 20250808095203.png]]

En realidad para encontrar este url nos damos cuenta que el atacante descargo para esto podemos filtrar por el protocolo http o https en este caso ya que el mismo ctf nos da una pista 
esto en linux se podria hacer de la siguiente maneraÑ

cat auth.log | grep "https"
cat auth.log | grep "http"  #Esto en caso de no conocer el origen de la decarga 

![[Pasted image 20250808120737.png]]

Al filtrar esto solo nos  queda esto.

![[Pasted image 20250808120841.png]] 


Ya con esto podemos conseguir el sherlock. 

Fin.
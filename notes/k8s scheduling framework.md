---
share: true
clean: true
title: Kubernetes Scheduling Framework
category: kubernetes/scheduler
tags: kubernetes
---
# Kubernetes Scheduling Framework
#kubernetes #scheduling 
## Intro
Basicamente añade una manera de extender el Scheduler. 
Los plugins se compilan con el Scheduler y le proveen de API adicional.
De esta manera el core del scheduler no cambia pero se pueden añadir nuevas funcionalidades.



## Workflow

El Framework define una serie de puntos donde los plugins pueden registrarse para ser llamados.

Los plugins pueden cambiar las decisiones del scheduler o bien simplemente ampliar la información disponible.

Todo intento de hacer scheduling de un nodo tiene dos fases diferenciadas:

- Scheduling Cycle
- Binding Cycle

A esto se le llama **scheduling context**

![[Kubernetes Scheduling Framework Extension Points.png]]

### Scheduling Cycle & Binding Cycle.

- Scheduling Cycle
  - Es el proceso de elegir un [[Kubernetes Node|nodo]] a un [[Kubernetes Pod|Pod]]
  - Se ejecuta en serie ( es decir hasta que no terminamos un pod no empezamos con el siguiente)
  - Es alimentado por **una cola ordenada de pods** esperando ser asignados a un nodo.
- Binding Cycle
  - Es donde se aplica la decision anterior al cluster
  - Se pueden ejecutar en paralelo.

### Extension Points
- [[#Scheduling Cycle|Scheduling Cycle]]
  - [[#QueueSort|QueueSort]]
  - [[#PreFilter|PreFilter]]
  - [[#Filter|Filter]]
  - [[#Post Filter|Post Filter]]
  - [[#Pre Score|Pre Score]]
  - [[#Score|Score]]
  - [[#NormalizeScore|NormalizeScore]]
  - [[#Reserve|Reserve]]
  - [[#Permit|Permit]]
- [[#Binding Cycle|Binding Cycle]]
  - [[#PreBind|PreBind]]
  - [[#Bind|Bind]]
  - [[#PostBind|PostBind]]

#### Scheduling Cycle


##### QueueSort
- Ordenan los Pods en la cola de espera
- Solo puede haber un plugin de este tipo activo al mismo tiempo

##### PreFilter
- Comprueban cierta informacion sobre el Pod y condiciones que tanto el pod como el cluster tienen que cumplir.
- Si uno de los PreFilter Plugins retorna error se aborta el Scheduling Cycle.

##### Filter
- Se usa para Filtrar los Nodos que **NO** pueden ejecutar el pod.
- Para cada nodo se ejecutan todos los plugins en el orden configurado
- Si un filtro marca un nodo como no válido no se ejecutan los demas en dicho nodo 
  - Es decir si ya sabemos que un nodo no es valido no tiene sentido continuar
- Los nodos pueden ser evaluados de manera **concurrente**

##### Post Filter
- Estos plugins se llaman despues de la fase de "Filter" pero **solo cuando ningun nodo ha pasado el filtrado** ( es decir que no tenemos ningún nodo válido para poder meter el pod.)
  - Aqui sería el momento de eliminar pods de un nodo para hacer sitio al nuevo de ser necesario. --> [[Kubernetes scheduler Pod Priority and Preemption]]
- Se llama a todos los plugins en el orden configurado.
- Si cualquier plugin marca un nodo como `Schedulable` (valido) el resto de plugins no se llaman.
  - Supongo que, una vez encuentra uno donde encajar el pod no sigue porque se supone que estas buscando el "menos malo"

##### Pre Score
- Sirven para hacer un pre-score de los pods, y luego poder compartir esa informacion con el resto de los plugins de Score.
- Si uno de estos plugins retorna un error se aborta el Scheduling Cycle

##### Score
- Se utiliza para dar una puntuacion ( scoring) a todos los nodos que han pasado el filtro.
- Se llama a cada uno de los plugins de score para cada uno de los nodos.
- Hay un rango definido de enteros para representar el score de un nodo.
- Despues de la fase de Normalizacion el scheduler **combinará la puntuacion de todos los plugins para un nodo segun el peso que cada plugin tenga configurado**

##### NormalizeScore
- Sirven para adaptar la puntuacion dada por un Score plugin al intervalo definido.
- Un plugin que se registra para esta extension **solo será llamado con la información que el mismo generó en la fase de Score**
- Esto se llama una vez por plugin y por scheduling cycle.
- Si un plugin de estos retorna un error se aborta el Scheduling cycle.

##### Reserve
- Los plugins que se registren aqui tienen que tener dos metodos `Reserve` & `Unreserve`.
- Esto se llama antes de que empiece la fase de Binding para evitar condiciones de carrera.
  - Basicamente se les da la oportunidad a los plugins de "reservar" ciertos recursos de un nodo para que no sean tenidos en cuenta en los subsiguientes Scheduling cycles aun cuando no se ha producido realmente el binding, para evitar que haya varios Pods luchando por los mismos recursos.
- El metodo `Reserve` puede fallar. Si un plugin falla no se llama al resto de los plugins y se considera que la fase ha fallado, abortando el Scheduling cycle.
- `Unreserve` se llama si falla la fase de Reserva en un plugin posterior o si falla alguna de las fases posteriores a la de Reserve.
  - Cuando esto pasa se llama al `Unreserve` de ==todos== los plugins en el orden inverso al de las llamadas a `Reverse`.
    ````ad-warning
    La implementacion de los `Unreserve` tiene que ser ==idempotente== y no puede fallar.
    ````

##### Permit
- Es el ultimo punto de ejecucion en el Scheduling Cycle. 
- Permite denegar un scheduling o retrasarlo.
- Un permit plugin puede hacer una de las **tres** siguientes cosas:
  - **approve**: Se envía al  Binding Cycle inmediatamente.
  - **deny**: Se cancela el Binding y se envia el pod de nuevo a la cola de espera para Scheduling.
  - **wait** (with timeout) :  
    - Envía el pod a una cola de **waiting**.
    - El Binding Cycle del pod comienza pero **queda inmediatamente bloqueado hasta ser aprovado**
    - Si ningún plugin lo aprueba antes de que pase el timeout el `wait` se convierte en `deny` y el Pod se devuelve a la cola de espera para el Scheduling.

#### Binding Cycle

##### PreBind
- Aqui se debe hacer cualquier trabajo preliminar antes de permitir que el pod se ejecute en el nodo
  - Ej: Montar un volumen de red
- Si cualquiera de los plugins retorna un error el pod es `rejected` y enviado a la cola de Scheduling

##### Bind
- Se encargan de hacer el bind de un pod a un nodo ( no tengo ni idea de que implica esto ahora mismo.)
- Se llama a cada plugin en el orden configurado
- No son llamados hasta que no han terminado los de PreBinding
- Un plugin puede elegir si maneja o no un determinado Pod.
- En el caso de que un plugin elija manejarlo **no se sigue llamando a los demas para ese pod**


##### PostBind
- Esto es basicamente para informar de que el proceso ha terminado
- Puede utilizarse para hacer limpieza de recursos utilizados en el proceso.


## Links
[official k8s doc](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/)

---
share: true
clean: true
title: Kubernetes Pod Priority and Preemption
category: kubernetes/scheduler
tags: kubernetes
---
# Pod Priority and Preemption

#kubernetes #scheduling #preemption 
Links:
- [main](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)

## Info
> [!info] **FEATURE STATE:** `Kubernetes v1.14 [stable]`

Los [[Kubernetes Pod|Pod]] tienen una prioridad.
Esta prioridad indica la *importancia relativa que tiene un pod con respecto a los demás*
Si un [[Kubernetes Pod|Pod]] no puede ser asignado a un nodo el [[Kubernetes Scheduler|scheduler]] intenta eliminar (evict) pods que tengan menos prioridad para poder asignar el pod.

## How to use it.

1. Crear una o mas [[Kubernetes PriorityClass]]
2. Crear pods con el campo`priorityClassName` seteado a alguno de los nombres de alguna de las [[Kubernetes PriorityClass|PriorityClass]] que hayamos definido. 

![[Kubernetes PriorityClass|PriorityClass]]

![[Kubernetes Non-preempting PriorityClass]]

## Workflow
Cuando un [[Kubernetes Pod|Pod]] con [[Kubernetes PriorityClass|PriorityClass]] entra el `priority admission controller` busca la clase y popla el campo prioridad con el valor entero.
Cuando las prioridades están activadas, *la cola de pods esperando scheduling se ordena por prioridad* para que los pods con mayor prioridad sean asignados antes.
Como existe el [[Kubernetes scheduler back-off]] todavia es posible que un nodo con menor prioridad pero con menos requisitos de recursos, sea asignado a un nodo antes que otro pod de mayor prioridad pero con unos requisitos de recursos mayores que no se pueden cumplir.

### Preemption 
[more info](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#preemption)
Los pods pendientes de scheduling se meten en una cola a esperar.
El scheduler toma un pod de la cola e intenta asignarlo a un [[Kubernetes Node]]
Si ningún nodo satisface los requisitos del [[Kubernetes Pod|Pod]] *se lanza la logica de preemption para dicho pod*

> [!important] Definicion: Logica de Preemption
Supongamos que el Pod es `P`.
La lógica de preempcion intenta en contrar un Nodo `N` tal que la eliminación de uno o mas Pods de menor prioridad que `P` permita que se satisfagan los requisitos de `P` y pueda ser asignado al nodo `N`
Si se encuentra un nodo que cumpla esto los pods de menor prioridad son sacados del nodo para que *mas tarde* `P` sea asignado al nodo.

Cuando se encuentra un nodo `N` donde eliminando pods de menor prioridad podemos meter a `P` *el nombre del nodo se mete en un campo del pod* llamado `nominatedNodeName`.
Esto es así porque el pod `P` no es asignado a `N` directamente, si no que se lanza la operacion de eliminacion de los pods de `N` para hacer sitio a `P` y esta operación puede tardar un tiempo ( porque los pods pueden tener tiempos de *graceful termination* por ejemplo), asi que lo que se hace es *retrasar la operacion de asignacion de `P`*, pero cuando se retoma se tiene en cuenta el valor del campo`nominatedNodeName` como nodo **preferente**

> [!caution] `P` no tiene porque ser asignado **necesariamente** al nodo en `nominatedNodeName`

El scheduler siempre intenta primero el `nominatedNodeName` antes de iterar por los demas nodos en busca de alguno pero, como la operacion de asignacion de `P` no se sincroniza con la de eliminación de los otros pods de `N`, puede pasar que entre medias haya otro nodo que se libere de recursos y donde podamos asignar a `P`
Es decir `nominatedNodeName` y `nodeName` no tienen porque tener el mismo valor.
Tambien puede pasar que, una vez liberado el espacio de `N` llegue un pod `P1` con una prioridad mayor que la de `P` y el scheduler le asigne antes al nodo `N` ocupando los recursos que en principio se liberaron para `P`. 
En este caso el scheduler borra el valor del campo `nominatedNodeName` de `P`, *esto hace que `P` pueda ser elegible para lanzar la eliminacion de pods en otro nodo* 


[[Kubernetes Preemption Evaluator and Interface]]
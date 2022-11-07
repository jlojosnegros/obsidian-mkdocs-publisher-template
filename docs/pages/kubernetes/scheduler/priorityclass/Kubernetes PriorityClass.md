---
share: true
aliases: ["PriorityClass", "Priority Class"]
category: kubernetes/scheduler/priorityclass
---

# PriorityClass
#kubernetes #scheduling 

- Es un objeto **sin namespace** es decir **clusted scoped**
- Sirve basicamente para hacer un mapping entre un nombre y un valor de prioridad.
- **Cuanto mayor el valor mayor la prioridad**
- El valor de la prioridad puede ser cualquier *entero de 32-bits* ==menor o igual a 1 billon== 
  - los valores por encima de estos estan **reservados** para pods criticos del sistema
- Tiene dos campos opcionales:
  - `description`: Que no es mas que una cadena human readable para los usuarios
  - `globalDefault`: `bool` Si es `true` será la prioridad que se les asignará a los pods que no tengan 
    - Logicamente: *solo un objeto `PriorityClass` puede tener este valor a `true` al mismo tiempo*
    - Si ninguna lo tiene a `true` se asigna **cero** como valor por defecto de la prioridad.

> [!example]- PriorityClass
> ```yaml
> apiVersion: scheduling.k8s.io/v1
> kind: PriorityClass
> metadata:
>   name: high-priority
> value: 1000000
> globalDefault: false
> description: "This priority class should be used for XYZ service pods only."
> ```




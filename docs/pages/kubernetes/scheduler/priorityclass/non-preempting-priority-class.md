---
aliases: ["Non-preempting PriorityClass"]
share: true
---

# Non-preemptive PriorityClass

> [!info] **FEATURE STATE:** `Kubernetes v1.24 [stable]`

[info source](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#non-preempting-priority-class)

Hay un campo dentro de la [[Kubernetes PriorityClass|PriorityClass]] que determina la politica de "preemtion" de un [[Kubernetes Pod|Pod]] ... vamos a quien puede el pod echar de un nodo para poder meterse el.
El campo es `preemptionPolicy`
El valor por defecto es `PreemptLowerPriority`

Cuando el valor del campo es `Never`:
- El pod se pone delante de los pods con menor prioridad en la cola de scheduling ( que entiendo que es para lo que sirve la prioridad)
- El pod **no puede hacer que ninguno otro pod sea echado del nodo para poder entrar el**
- Como todos los demas pods, uno con esta clase está sujeto a *scheduler back-off*: ![[Kubernetes scheduler back-off]]
- Evidentemente, los non-preemtive pods aun pueden ser echados de un nodo por otro nodo de mayor prioridad ( y que no tenga esta policita claro)

> [!example]- Non-Preemptive PriorityClass
>```yaml
> apiVersion: scheduling.k8s.io/v1
> kind: PriorityClass
> metadata:
>   name: high-priority-nonpreempting
> value: 1000000
> preemptionPolicy: Never
> globalDefault: false
> description: "This priority class will not cause other pods to be preempted."
>```

---
title: Kubernetes scheduler back-off
---
Esto es que si no hay recursos para poder meterlo en otro nodo se devuelve a la cola para poder intentarlo despues de un rato a ver si hay mas suerte. Estos reintentos cada vez se hacen con una menor frecuencia y entre medias es posible que otro pod de menor prioridad pero que necesita menos recursos pueda ser asignado a un nodo, con lo que se puede dar una especie de "inversion de prioridad"

#kubernetes #scheduling 
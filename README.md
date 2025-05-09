# Laboratorio 04

> Elaborado por: Sebastián Chávarry

Una startup de desarrollo necesita una herramienta para organizar su operación semanal. El sistema debe permitir registrar tareas, bugs y solicitudes de features, cada uno con título, descripción, asignado a, fecha de creación y un estado (completed o resolved). Los usuarios deben poder crear, asignar, marcar como resueltos y deshacer las últimas acciones. Además, los líderes del equipo quieren poder recorrer listas filtradas (como solo tareas pendientes o bugs resueltos), sin acceder directamente a las estructuras internas.

## Preguntas

1. ¿Qué clases concretas deben modelarse para representar los elementos del sistema?

2. ¿Cómo pueden encapsular cada acción del usuario (crear, asignar, completar) para luego poder deshacerla?

3. ¿Cómo estructurarían la lógica para recorrer solo ciertos elementos (por ejemplo, solo tareas no completadas), sin usar `for` externos o exponer listas?

4. ¿Cómo harían que el sistema permita en el futuro agregar otros tipos de elementos sin romper el diseño actual?

5. ¿Cómo implementarían la función `undo()` sin romper el principio de responsabilidad única?

<img width="882" height="443" alt="image" src="https://github.com/user-attachments/assets/6a9c36f2-2543-4c88-88c4-8b6b4085cd01" />

# Arquitectura de red

1. Arquitectura de Red y Diseño del Escenario
Para dar inicio a este laboratorio, es fundamental comprender la topología de red diseñada. A diferencia de entornos de explotación lineal o de baja dificultad, este escenario presenta una arquitectura segmentada compuesta por cuatro máquinas interconectadas y distribuidas a través de cuatro subredes lógicas distintas.

<img width="1567" height="672" alt="image" src="https://github.com/user-attachments/assets/fcb5e26c-7280-42b5-9099-5256ff562224" />

Segmentación y Conectividad (Dual-Homed)
La característica crítica de esta infraestructura es que cada nodo intermedio está configurado como una máquina dual-homed. Esto implica que cada activo posee al menos dos interfaces de red físicas o virtuales activas, permitiéndole coexistir en dos segmentos de red simultáneamente.

Esta configuración es el pilar que permite el movimiento lateral, ya que estas máquinas actúan como "puentes" naturales (gateways) entre zonas que, de otro modo, estarían completamente aisladas entre sí.

`Referencia Visual de la Infraestructura
Para facilitar el seguimiento de la intrusión y mantener una trazabilidad clara del progreso, se ha definido el siguiente esquema. En él se detallan los direccionamientos IP y nombres de host orientativos, los cuales servirán como hoja de ruta a lo largo de esta guía:
`
# 2. Objetivos Estratégicos y Metodología
El propósito central de este laboratorio es el perfeccionamiento de técnicas de Pivoting. El ejercicio demanda que el operador sea capaz de tunelizar el tráfico desde una red externa hacia segmentos profundos de la infraestructura interna, utilizando cada máquina comprometida como un punto de salto (jump server).
`
El Flujo del Compromiso
El éxito del ejercicio se define por la capacidad de comprometer la integridad de todos los activos, progresando de forma escalonada desde la RED 1 hasta alcanzar el objetivo final en la RED 4.
`
Nota sobre la Escalada de Privilegios
Durante la fase de post-explotación en cada nodo, se realizaron auditorías exhaustivas mediante la enumeración de vectores comunes (scripts de automatización, búsqueda de capabilities, SUID, y vulnerabilidades de kernel). Tras el análisis, se determinó lo siguiente:

Enfoque en Movimiento Lateral: No se identificaron vectores de escalada de privilegios críticos o intencionados en los nodos intermedios.

Diseño del Reto: El entorno parece estar optimizado específicamente para la práctica de enrutamiento y gestión de proxies (como Chisel, Ligolo-ng o Metasploit autoroute).

Nota de edición: Con el fin de mantener la guía concisa y enfocada en el aprendizaje del pivoting, se han omitido los procesos de enumeración negativa para la escalada de privilegios, procediendo directamente al establecimiento de túneles tras la obtención del acceso inicial en cada máquina.

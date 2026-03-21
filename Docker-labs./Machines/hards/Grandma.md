<img width="882" height="443" alt="image" src="https://github.com/user-attachments/assets/6a9c36f2-2543-4c88-88c4-8b6b4085cd01" />

# 🧠 Laboratorio de Pivoting y Movimiento Lateral

## 🏗️ Arquitectura de Red y Diseño del Escenario

Para dar inicio a este laboratorio, es fundamental comprender la topología de red diseñada. A diferencia de entornos de explotación lineal o de baja dificultad, este escenario presenta una arquitectura segmentada compuesta por **cuatro máquinas interconectadas**, distribuidas a través de **cuatro subredes lógicas distintas**.

![Arquitectura de Red](https://github.com/user-attachments/assets/fcb5e26c-7280-42b5-9099-5256ff562224)

---

## 🔀 Segmentación y Conectividad (Dual-Homed)

La característica crítica de esta infraestructura es que cada nodo intermedio está configurado como una máquina **dual-homed**.

Esto implica que cada activo posee al menos **dos interfaces de red** (físicas o virtuales) activas, permitiéndole:

- Coexistir en múltiples segmentos de red simultáneamente  
- Actuar como puente entre subredes  
- Facilitar el movimiento lateral  

💡 Esta configuración es el pilar del laboratorio, ya que estas máquinas funcionan como **gateways naturales** entre zonas que normalmente estarían completamente aisladas.

---

## 🗺️ Referencia Visual de la Infraestructura

Para facilitar el seguimiento de la intrusión y mantener una trazabilidad clara del progreso, se ha definido un esquema donde se detallan:

- Direccionamientos IP  
- Nombres de host  
- Relación entre segmentos de red  

Este esquema servirá como **hoja de ruta** a lo largo de toda la guía.

---

## 🎯 Objetivos Estratégicos y Metodología

El propósito central de este laboratorio es el perfeccionamiento de técnicas de **Pivoting**.

El ejercicio demanda que el operador sea capaz de:

- Tunelizar tráfico desde una red externa  
- Acceder a segmentos internos progresivamente  
- Utilizar cada máquina comprometida como un **jump server**  

---

## 🔄 Flujo del Compromiso

El éxito del ejercicio se define por la capacidad de comprometer todos los activos de forma escalonada:


RED 1 → RED 2 → RED 3 → RED 4


Cada sistema comprometido permite:

- Expandir el acceso dentro de la red  
- Establecer nuevos túneles  
- Alcanzar segmentos previamente inaccesibles  

---

## 🔐 Nota sobre la Escalada de Privilegios

Durante la fase de post-explotación en cada nodo, se realizaron auditorías exhaustivas mediante:

- Scripts de enumeración automatizada  
- Búsqueda de capabilities  
- Identificación de binarios SUID  
- Análisis de vulnerabilidades del kernel  

### 📌 Resultados

- ❌ No se identificaron vectores de escalada de privilegios críticos o intencionados  
- 🎯 El entorno está enfocado en el **movimiento lateral**  

---

## 🧩 Diseño del Reto

El laboratorio está optimizado específicamente para la práctica de:

- Pivoting  
- Enrutamiento interno  
- Gestión de túneles y proxies  

### 🔧 Herramientas recomendadas

- Chisel  
- Ligolo-ng  
- Metasploit (autoroute)  

---

## ✍️ Nota de Edición

Con el fin de mantener la guía concisa y enfocada en el aprendizaje del pivoting:

> Se han omitido los procesos de enumeración negativa relacionados con la escalada de privilegios.

Tras obtener acceso inicial en cada máquina, se procede directamente al:

- Establecimiento de túneles  
- Movimiento lateral hacia el siguiente objetivo  

# 🛡️ Simulación de Ataque DoS y Monitoreo en Tiempo Real

## 📝 Descripción
Este proyecto consiste en la implementación de un entorno de laboratorio controlado para simular ataques de Denegación de Servicio (DoS) en la Capa 7 (Aplicación). El objetivo es analizar el impacto directo del tráfico malicioso sobre los recursos críticos del sistema (CPU y Red) utilizando herramientas de monitoreo profesional.

## 🛠️ Stack Tecnológico
- **Orquestación de Red:** Emulador de red basado en web desarrollado por Ariel Villalobos. ([https://github.com/RArielVillalobos/open-veth]).

- **Entorno:** Contenedores Docker vía OpenVeth.

- **Atacante (HOST):** Scripts de automatización en Bash y cURL.

- **Víctima (SERVER):** Servidor Nginx monitoreado.

- **Monitoreo:** Prometheus (Recolección de datos) y Grafana (Visualización).

## 🚀 Metodología del Experimento
1. **Configuración del Dashboard**
   
  Se diseñaron dos paneles de control en Grafana utilizando lenguaje PromQL:

  * Métrica de Red: Visualización de tráfico entrante en eth0 mediante irate.

  * Métrica de CPU: Cálculo del porcentaje de uso real, restando el tiempo de inactividad (idle state).

2. **Ejecución del Ataque**
   
  Se utilizó un script de persistencia en el nodo atacante para generar un flujo masivo de peticiones HTTP:

`` 
for i in {1..10}; do (while true; do curl -s 172.17.0.5 > /dev/null; done &); done
``

*Este comando lanza múltiples procesos en segundo plano, garantizando que el tráfico no se detenga incluso si una petición individual falla.*

3. **Fase de Mitigación y Persistencia**
   
  Durante el experimento, se intentó mitigar el ataque mediante killall curl. Se observó un fenómeno de persistencia, donde los procesos "padre" (bucles while) regeneraban el tráfico instantáneamente, manteniendo la CPU al   100% de saturación.

## 📊 Resultados Observados

**Correlación Crítica:** Se grabó en video cómo el incremento súbito en la tasa de transferencia de red (Bytes/sec) dispara el consumo de CPU de forma casi inmediata, llevando al servidor a un estado de Resource Exhaustion (Agotamiento de Recursos).

**Estado Base:** CPU < 1%, Red 0 bps.

**Bajo Ataque:** CPU 85-100%, Red en picos constantes.

**Recuperación:** Requirió un reinicio de los servicios (docker restart) para limpiar los procesos huérfanos y estabilizar las métricas.

## 🧠 Lecciones Aprendidas
**Visibilidad:** Sin monitoreo (Grafana), un ataque DoS es invisible hasta que el servicio cae; con él, podemos reaccionar en segundos.

**Persistencia:** Matar el proceso final (curl) no siempre es suficiente si el atacante usa scripts de automatización.

**Impacto de Capa 7:** No se necesita un ancho de banda masivo para tumbar un servidor; basta con saturar su CPU procesando peticiones legítimas.

## 📂 Archivos Adjuntos
- *Grabación en tiempo real* del dashboard durante el ataque.
  ![grafana-charts](https://github.com/user-attachments/assets/3c3ea8cf-98e1-4c38-8499-18f5c892222e)


- Captura de pantalla de dashboard para Trafico de red (Capa 7)
<img width="1196" height="521" alt="grafana_capture" src="https://github.com/user-attachments/assets/0d0a627f-3b98-42ab-b17a-8d3ad56be215" />


  
  


  

## Parcial-2

<img width="1047" height="590" alt="image" src="https://github.com/user-attachments/assets/abc99b7f-6dbf-458b-8265-258de295aee9" /><br>
🔴***Diferencia entre NetFlow y sFlow:*** <br>
**NetFlow:** Realiza un seguimiento de todos los paquetes pertenecientes a un flujo específico. Es más detallado pero consume más recursos de procesamiento<br>
**sFlow:** Es un muestreo estadístico basado en paquetes; no analiza cada paquete sino una muestra (1 de cada N)<br>
**Elección para 100 Gbps:** Elegiría sFlow, ya que a velocidades tan altas como 100 Gbps, procesar cada paquete para NetFlow puede saturar la CPU del router; el muestreo estadístico de sFlow es mucho más eficiente para identificar "top talkers" sin degradar el rendimiento del enlace<br><br>

🔴***Los 5 campos de la 5-tuple en NetFlow:*** <br>
**Dirección IP de Origen (Source IP):** Identifica quién envía los datos.<br>
**Dirección IP de Destino (Destination IP):** Identifica hacia dónde van los datos.<br>
**Puerto de Origen (Source Port):** El puerto en el dispositivo de origen (a menudo efímero).<br>
**Puerto de Destino (Destination Port):** El puerto en el dispositivo de destino que identifica el servicio (ej. 80, 443).<br>
**Protocolo de Capa 4 (Protocol):** Ej. TCP (6) o UDP (17).<br><br>

🔴***Medición por aplicación:*** <br>
Para diferenciar HTTP de SSH, el colector debe inspeccionar el puerto de destino, un ejemplo de este puede ser el puerto 80/443 para HTTP y puerto 22 para SSH, se debe tener en cuenta que el puerto destino es el indicador clave para identificar la naturaleza de la aplicación.<br><br>

🔴***Interpretación de datos (IP Accounting):*** <br> 
La tabla muestra el tráfico que pasa a través del enrutador de Cisco. Los recuentos de paquetes y bytes son los totales para cada par de origen-destino acumulados hasta el momento. No se trata de flujos completados, sino de un contador en tiempo real. Los datos muestran flujos entre 192.168.1.10 y 10.0.0.5.<br>
Se observa que el primer host envió 1500 paquetes, mientras que el segundo solo respondió con 50.<br><br>

🔴***Asimetría extrema:*** <br>
Una diferencia tan marcada indicaría que la comunicación es mayormente unidireccional, lo que podría sugerir una transferencia masiva de datos (descarga), un posible ataque de denegación de servicio o problemas de enrutamiento asimétrico en la red.<br>
La asimetría extrema (como en este caso, 1500 paquetes enviados vs. 50 recibidos) entre 192.168.1.10 y 10.0.0.5 indicaría típicamente:<br>
**Comportamiento de Carga Masiva (Bulk Upload):** El host .1.10 está cargando datos masivos a un servidor en .0.5. El tráfico de ida es pesado, mientras que el tráfico de vuelta son solo confirmaciones (ACKs) de TCP muy pequeñas.<br>
**Ataque de DoS/DDoS (ej. Inundación SYN):** Un host puede estar enviando una gran cantidad de solicitudes de conexión (paquetes SYN) a un servidor, y el servidor no responde (o las respuestas no vuelven a través de este enrutador), creando una enorme disparidad.<br>
**Escaneo de Red/Puertos:** Un host escaneando miles de puertos en otro host, y solo unos pocos puertos responden.<br>
<img width="1024" height="545" alt="image" src="https://github.com/user-attachments/assets/0de61598-e557-476d-a369-0a5fb989f704" /><br><br>

<img width="1046" height="517" alt="image" src="https://github.com/user-attachments/assets/778fb8d1-609c-48a2-891f-9678caca20d4" /><br>
```mermaid
graph LR
    A[Cámara USB] --> B(Contenedor Docker: YOLOv8)
    B -- "Tráfico UDP/TCP" --> C(VM: softflowd)
    C -- "Flujos NetFlow" --> D(Colab: nfdump)
    D --> E[Dashboard: Matplotlib/Streamlit]
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#bbf,stroke:#333,stroke-width:2px
   ```
Para que el contenedor YOLO se comunique con la VM y el tráfico sea muestreado:<br>
🔵**Comunicación:** Se debe configurar una red virtual (como un bridge de Docker o un switch virtual) donde el contenedor tenga una IP en el mismo segmento que la VM.<br><br>
🔵**Regla de IP Accounting (iptables):** Para medir el tráfico, usaría: iptables -A FORWARD -s 172.17.0.0/16 -d 192.168.1.10 -j ACCEPT ---> Para medir tráfico saliente del contenedor a la VM o podría usar también iptables -A FORWARD -s 192.168.1.10 -d 172.17.0.0/16 -j ACCEPT ---> Para medir tráfico entrante de la VM al contenedor, también podría revisar las estadisticas con iptables -L FORWARD -v -n. <br><br>
🔵**Descripción diagrama de flujo**<br>
*Cámara USB → Contenedor YOLOv8:* La cámara captura el video raw y lo entrega al contenedor (usualmente montando el dispositivo /dev/video0).<br>
*YOLOv8 → Red (Tráfico UDP/TCP):* YOLO procesa las detecciones y envía los resultados (metadatos o video procesado) a través de la red hacia un destino. Esto genera paquetes IP.<br>
*Tráfico → VM (softflowd):* Los paquetes pasan por la interfaz de red de la VM. Aquí, softflowd (el exportador) observa el tráfico, extrae la 5-tupla y crea paquetes NetFlow.<br>
*softflowd → Colab (nfdump):* La VM envía los "Flujos NetFlow" (paquetes UDP, usualmente puerto 2055) hacia la instancia de Google Colab donde corre nfdump.<br>
*nfdump → Dashboard:* nfdump procesa los datos binarios de NetFlow, los convierte a un formato legible (CSV o JSON) y Matplotlib/Streamlit lee esos datos para graficar el consumo de ancho de banda en tiempo real.<br><br>

<img width="1022" height="540" alt="image" src="https://github.com/user-attachments/assets/f3f33011-137e-4bcc-aed1-1afb7eeafbc8" /><br>
<img width="1024" height="547" alt="image" src="https://github.com/user-attachments/assets/0f4d76a1-0ee4-4c1a-a0ea-11102357bf27" /><br>





```mermaid
graph TD
    subgraph Red_Estacion [Subred 10.0.0.0/24]
        C1[C1: Placas] 
        C2[C2: Parqueo]
        C3[C3: Aforo]
        C4[C4: Animales]
        C5[C5: Objetos]
    end
    
    C1 & C2 & C3 & C4 & C5 --> SW[Switch Virtual / Bridge]
    SW --> VM1[VM1: Colector Principal]
    SW --> VM2[VM2: Respaldo Redundante]
    
    style VM1 fill:#dfd
    style VM2 fill:#fdd
```

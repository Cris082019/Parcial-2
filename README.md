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
Para el diagrama descrito en la imagen usamos contenedores Docker (uno por cámara) que ejecutan modelos YOLOv8. Cada uno tiene una misión distinta: leer placas, contar personas o detectar objetos olvidados. De aquí salen dos cosas: el video con etiquetas y los datos puros (metadata).<br><br>

🟢***Red y NetFlow*** <br>
Para enviar esta información al centro de datos, usamos un Router Virtual. Este componente hace dos tareas clave:<br>
**Vigila el tráfico (NetFlow):** Mide cuánto ancho de banda consume cada cámara para que la red no se sature.<br>
**Prioriza lo importante (QoS):** Si la red está lenta, el sistema da paso preferencial a las alertas de seguridad (metadata) por encima del video común.<br>

🟢***Servidores Centrales*** <br>
La información llega a un grupo de servidores redundantes. Esto significa que, si un servidor falla, los otros toman el control de inmediato para que el monitoreo nunca se detenga. Aquí se guarda todo en una base de datos segura y se organizan los reportes de tráfico.<br><br>

🟢***Dashboard***<br>
Finalmente, toda esa tecnología se traduce en algo que una persona puede entender fácilmente:<br>
**Seguridad:** Ve las alertas en tiempo real y el video de las cámaras.<br>
**Ingeniería:** Revisa gráficas claras (hechas en Streamlit) sobre la salud de la red y el consumo de datos gracias al análisis de NetFlow.<br><br>

<img width="1023" height="490" alt="image" src="https://github.com/user-attachments/assets/6dc84aea-5f4c-4bb6-b59e-e8e49905d175" />
<img width="1028" height="443" alt="image" src="https://github.com/user-attachments/assets/e11dacd2-cb21-446d-ac76-7d1d6f7b9d5f" /><br><br>

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
➖***Cálculo de Throughput por contenedor:*** <br>
**Video:** 30 fps × 50 KB/frame = 1500 KB/s. Convertido a bits: 1500×8=12,000 Kbps o 12 Mbps<br>
**Metadata:** 200 bytes × 10 det/s = 2000 bytes/s. Convertido a bits: 2000×8=16,000 bps o 0.016 Mbps<br>
**Total por contenedor:** 12.016 Mbps.<br>
**Total 5 contenedores:** 60.08 Mbps<br><br>

➖***Protocolo para Video:*** <br>
El más adecuado es UDP. Debido a que el video es en tiempo real y la red tiene baja latencia (2 ms), es mejor perder un frame ocasional que retrasar todo el stream esperando retransmisiones de TCP<br><br>

➖***Mitigación de Jitter:*** <br>
Se debe implementar un jitter buffer en el receptor para almacenar temporalmente los paquetes y entregarlos de manera constante.<br><br>

Cuando hablamos de las reglas de Netflow, se debe tener en cuenta la IP de origen específica de cada contenedor dentro de la subred 10.0.0.0/24, por otro lado, el IP Accounting es como un contador de luz: solo le importa cuánta energía (bytes) pasó por un punto, en este orden de ideas se usuaria de la siguiente forma, se debe activar en la interface del Switch Virtual o el Router, este lo que hace es guardar una tabla dinamica con IP Origen, IP Destino, Bytes y Paquetes, para detectar el mayor emisor se limpia la tabla (clear), se deja correr por 5 minutos, se ejecuta el comando (ejemplo: show ip accounting), ahora lo que se puede observar es que se ordena por la columna de Bytes, se determina que el contenedor con la IP que tenga el número mas alto es el "top talker"<br><br>

<p align="center"><img width="45%" height="580" alt="image" src="https://github.com/user-attachments/assets/776ac456-03a9-478d-91a2-9afc92b0b5c6" /><img width="45%" height="538" alt="image" src="https://github.com/user-attachments/assets/b07786f6-2221-4019-8fe9-91d293c6dcd0" /><img width="45%" height="534" alt="image" src="https://github.com/user-attachments/assets/100ea5cb-abf8-4135-b8c0-ed9c99b1969b" /><img width="45%" height="475" alt="image" src="https://github.com/user-attachments/assets/83ef7f68-3ede-403b-9318-af7fb89cf7a9" /><img width="45%" height="465" alt="image" src="https://github.com/user-attachments/assets/126c5ef7-fe5f-481e-826f-02771e0a3f51" /><img width="45%" height="467" alt="image" src="https://github.com/user-attachments/assets/4ed83768-63f5-4cbb-9ab0-68e557f76a2e" /></p> 






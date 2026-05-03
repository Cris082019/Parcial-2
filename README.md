## Parcial-2

<img width="1047" height="590" alt="image" src="https://github.com/user-attachments/assets/abc99b7f-6dbf-458b-8265-258de295aee9" /><br>
***Diferencia entre NetFlow y sFlow:***
**NetFlow:** Realiza un seguimiento de todos los paquetes pertenecientes a un flujo específico. Es más detallado pero consume más recursos de procesamiento<br>
**sFlow:** Es un muestreo estadístico basado en paquetes; no analiza cada paquete sino una muestra (1 de cada N)<br>
**Elección para 100 Gbps:** Elegiría sFlow, ya que a velocidades tan altas como 100 Gbps, procesar cada paquete para NetFlow puede saturar la CPU del router; el muestreo estadístico de sFlow es mucho más eficiente para identificar "top talkers" sin degradar el rendimiento del enlace









```mermaid
graph LR
    A[Cámara USB] --> B(Contenedor Docker: YOLOv8)
    B -- "Tráfico UDP/TCP" --> C(VM: softflowd)
    C -- "Flujos NetFlow" --> D(Colab: nfdump)
    D --> E[Dashboard: Matplotlib/Streamlit]
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#bbf,stroke:#333,stroke-width:2px
   ``` 

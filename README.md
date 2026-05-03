## Parcial-2

graph LR<br>
    A[Cámara USB] --> B(Contenedor Docker: YOLOv8)<br>
    B -- "Tráfico UDP/TCP" --> C(VM: softflowd)<br>
    C -- "Flujos NetFlow" --> D(Colab: nfdump)<br>
    D --> E[Dashboard: Matplotlib/Streamlit]<br>
    style B fill:#f9f,stroke:#333,stroke-width:2px<br>
    style C fill:#bbf,stroke:#333,stroke-width:2px<br>

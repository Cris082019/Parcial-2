## Parcial-2

```mermaid
graph LR
    A[Cámara USB] --> B(Contenedor Docker: YOLOv8)
    B -- "Tráfico UDP/TCP" --> C(VM: softflowd)
    C -- "Flujos NetFlow" --> D(Colab: nfdump)
    D --> E[Dashboard: Matplotlib/Streamlit]
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#bbf,stroke:#333,stroke-width:2px
   ``` 

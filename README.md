## Parcial-2

```mermaid graph LR A[Cámara USB] --> B(Contenedor Docker: YOLOv8) B -- "Tráfico UDP/TCP" --> C(VM: softflowd) C -- "Flujos NetFlow" --> D(Colab: nfdump) D --> E[Dashboard: Matplotlib/Streamlit] ```

# Integración de reportes de laboratorio (Power Automate)

Automatización para generar, convertir, almacenar y distribuir reportes de laboratorio, y centralizar la recolección de datos desde Microsoft Forms hacia SharePoint/OneDrive/Email.

## Accesos (SharePoint)
1) Análisis de HC recuperado
2) Rechazo (con análisis)
3) Rechazo (sin análisis)
4) Análisis de sólidos

Este repo documenta el flujo **“Reporte de rechazo de unidad”** (PDF + notificación) y referencia los demás.

## Arquitectura (Mermaid)
```mermaid
flowchart LR
A[Forms - Nueva respuesta] --> B[Power Automate - Obtener detalles]
B --> C[Filtrar links e imágenes]
C --> D[SharePoint - Obtener archivos]
D --> E[Insertar imágenes base64 en HTML]
B --> F[Construir HTML]
F --> G[OneDrive - Guardar HTML]
G --> H[OneDrive - Convertir a PDF]
H --> I[SharePoint - Guardar PDF oficial]
I --> J[Email - Enviar desde buzón compartido]

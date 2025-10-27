# Integración de reportes de laboratorio (Power Automate)

Automatización para generar, convertir, almacenar y distribuir reportes de laboratorio, y centralizar la recolección de datos desde Microsoft Forms hacia SharePoint/OneDrive/Email.

Este repo documenta el flujo **“Reporte de rechazo de unidad”** (PDF + notificación) y referencia los demás.

## Arquitectura (Mermaid)
```mermaid
flowchart LR
A[Forms - Nueva respuesta] --> B[Power Automate - Obtener detalles]
B --> C[Filtrar links de imágenes]
C --> D[SharePoint - Obtener imágenes]
D --> E[Iniciar variable HTML]
E --> F[Insertar imágenes base64 en HTML]
G --> H[Finalizar variable HTML]
H --> I[OneDrive - Crear archivo HTML]
I --> J[OneDrive - Convertir a PDF]
J --> K[SharePoint - Guardar PDF oficial]
K --> L[Email - Enviar desde buzón compartido]

## Desarrollo

**Integración de reportes de laboratorio**

Mediante una sencilla página web de sharepoint se centralizó la disponibilidad de todos los enlaces relevantes, mejorando el acceso y la navegación.

IMG Shp

Los cuatro links
superiores de la web de laboratorio corresponden a reportes que se hacen desde
el laboratorio:

1. Reportar un análisis de hidrocarburo recuperado
2. Reportar un rechazo (con análisis de laboratorio)
3. Reportar un rechazo (sin análisis de laboratorio)
4. Análisis de sólidos

Para cada uno de los
reportes se crearon flujos automatizados de power automate que automatizan la
generación, conversión, almacenamiento y distribución de reportes y recolección
de datos. Los 3 primeros reportes contienen 2 flujos automatizados, uno para la
generación, conversión, almacenamiento y distribución de reportes y uno
adicional que colecta la información en una única lista de sharepoint, en el
caso del Analisis de sólido, solo se colectan los datos ya que no requiere
reporte.

Cada flujo
automatizado de reporte funciona de forma similar variando la información del
informe y los destinatarios del correo electrónico. A continuacion se muestra
el detalle de uno de ellos:

**Nombre del flujo: Reporte de rechazo de unidad**

Este flujo
automatiza la generación, conversión, almacenamiento y distribución de reportes
de rechazo de unidades en el laboratorio de la planta de tratamiento de
residuos oleosos. A continuación se detalla su funcionamiento completo:

**🔹 Disparador**

- Se activa al recibir una nueva respuesta en el formulario TRON de Microsoft Forms.

**🔹 Extracción de datos**

- Recupera todos los campos relevantes de la respuesta, incluyendo:
    - Ticket
    - Fecha y hora
    - Manifiesto
    - Reporte
    - Motivo de rechazo
    - Transportista
    - Equipo
    - Unidad
    - Origen
    - Resultados del análisis (densidad, % aceite, % agua, % sólidos)
    - Volumen rechazado

**🔹 Procesamiento de imágenes**

- Extrae una lista de links desde un campo del formulario.
- Filtra los que contienen "link" y accede a SharePoint para obtener las imágenes.
- Inserta las imágenes en el HTML como bloques codificados en base64.

**🔹 Generación del documento**

- Construye un documento HTML con formato profesional.
- Incluye encabezado, secciones informativas, resultados, imágenes y pie de página.

**🔹 Conversión y almacenamiento**

- Guarda el HTML en OneDrive.
- Lo convierte a PDF.
- Finalmente, almacena el PDF en una carpeta específica de SharePoint para reportes oficiales.

**🔹 Distribución por correo**

- Envía un correo desde un buzón compartido a una lista de destinatarios internos.
- El correo incluye los datos clave del rechazo y adjunta el reporte PDF generado.

Este flujo garantiza
una trazabilidad completa del rechazo, desde la captura en el formulario hasta
la distribución del reporte, todo de forma automatizada y estandarizada.

**Paso a paso del flujo**

**1. 🟢 Disparador: "Cuando se envía
una respuesta nueva"**

- Se activa cuando se recibe una nueva respuesta en un formulario de Microsoft Forms.
- El formulario está identificado por un ID
- Usa un webhook para escuchar respuestas en tiempo real.

📌
**Nota**: Aquí podrías agregar el nombre del formulario y
qué tipo de información recopila.

**2. 📥 Acción: "Obtener los
detalles de la respuesta"**

- Recupera los datos completos de la respuesta enviada.
- Utiliza el response_id generado por el disparador para acceder a los campos específicos del formulario.

**3. 📝 Acción: "Redactar"**

- Extrae un campo específico de la respuesta.
- Este campo contiene una lista separada por comas. Corresponde a los links de las imágenes que se adjuntan en el formulario.

**4. 🧮 Acción: "Inicializar
variable - IdList"**

- Crea una variable tipo array llamada IdList.
- Divide el contenido del campo redactado en elementos individuales usando split(). Para extraer las URLs de las imágenes

**5. 🔍 Acción: "Filtrar
matriz"**

- Filtra los elementos del array IdList para quedarse solo con aquellos que contienen la palabra "link".

**6. 🧪 Acción: "Inicializar
variable - fileContent"**

- Crea una variable vacía tipo array llamada fileContent.
- Se usará más adelante para almacenar contenido (imágenes o archivos).

**7. 🧾 Acción: "Inicializar
variable - html_doc"**

- Crea una variable tipo string llamada html_doc.
- Contiene una plantilla HTML completa para generar un reporte visual.
- Incluye estilos CSS para formato A4, encabezados, secciones, contenedores flexibles, y soporte para impresión.

**8. 📂 Acción: "Obtener archivos
(solo propiedades)"**

- Accede a una carpeta específica en SharePoint para obtener los archivos disponibles.
- Ruta: /Documentos compartidos/Aplicaciones/Microsoft Forms/Analisis/FotoImagen a adjuntar al mail. Esta carpeta contiene las imágenes cargadas mediante el Forms.

**9. 🔁 Acción: "Aplicar a cada
uno"**

- Itera sobre los elementos filtrados previamente (los que contienen "link").
- Dentro del bucle realiza varias acciones:

**a. 🧪 "Redactar_1" y
"Redactar_2"**

- Decodifica y limpia la URL del archivo.

**b. 📥 "Obtener contenido de
archivo mediante ruta de acceso"**

- Descarga el contenido del archivo desde SharePoint.

**c. 📄 "Obtener metadatos de
archivo mediante ruta de acceso"**

- Recupera el nombre y otros metadatos del archivo.

**d. 📦 "Anexar a la variable de
matriz"**

- Agrega el contenido del archivo a la variable fileContent.

**e. 🖼️ "Anexar a la variable de
cadena 2"**

- Inserta la imagen en el HTML como un bloque <img> codificado en base64.

📌
**Nota**: El bucle está creado para recibir y almacenar
varios archivos. Sin embargo, se limita el formulario a la carga de 1 solo
archivo con un máximo de 1.4Mb.

**10. 🧩 Acción: "Anexar a la
variable de cadena 2-copy"**

- Finaliza el documento HTML agregando el cierre de etiquetas y el pie de página.

**11. 🗂️ Acción: "Crear archivo
2"**

- Guarda el documento HTML en OneDrive.
- El nombre del archivo incluye el Manifiesto y la fecha de envío del formulario.

**12. 🔄 Acción: "Convertir un
archivo mediante una ruta de acceso"**

- Convierte el archivo HTML en formato PDF usando OneDrive.

**13. 🗃️ Acción: "Crear archivo"**

- Guarda el archivo PDF final en una carpeta específica de SharePoint.
- En esta carpeta se guardan las evidencias e informes realizados por el laboratorio para el control documental y el sistema de gestión de excelencia operacional.
- Ruta: /Documentos compartidos/General/LABORATORIO/1. Reportes/Reportes/Reportes Lab 2025.

**14. 📧 Acción: "Enviar un correo
electrónico desde un buzón compartido (V2)"**

- Envía un correo desde el buzón compartido: YBG4134@grupo.ypf.com.
- Destinatarios: múltiples correos de AESA, YPF y SENPER, incluyendo áreas como logística, supervisión y residuos.
- Asunto: “Rechazo de unidad [Unidad]”, donde la unidad se extrae del formulario.
- Cuerpo del mensaje:
    - Incluye detalles como unidad, pozo, equipo, manifiesto, transportista, motivo de rechazo y volumen rechazado.
    - Firma: “Laboratorio Planta TRON – Servicios ambientales AESA”.
- Adjunta el archivo PDF generado previamente con el reporte.
- El objetivo de este correo electrónico es notificar la unidad rechazada al área de logística para su gestión y otros stakeholders


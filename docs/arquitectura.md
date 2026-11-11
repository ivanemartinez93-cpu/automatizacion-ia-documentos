# Arquitectura actual del prototipo

Formulario local → Gemini: análisis de PDF sintético → código de validación → Gemini: propuesta de ruta → tabla de datos local n8n.

- Los dos pasos de modelo son llamadas con roles diferenciados; no se han probado agentes autónomos con herramientas.
- Las credenciales se almacenan en n8n, fuera del JSON publicado.
- Se probó un PDF corrupto y un fallo simulado del segundo modelo con envío a revisión humana.
- Se observó un nodo webhook en el editor, pero su persistencia y funcionamiento no se han verificado. El export publicado corresponde a la versión anterior.
- Antes de usar datos reales falta sanitización fiable previa al proveedor, protección de documentos, pruebas de fallos y almacenamiento, observabilidad por agente y acceso seguro. No es un despliegue productivo.

## Incidente: falla del segundo modelo

1. Mantener el caso pendiente de revisión humana; no convertir una propuesta del modelo en una acción automática.
2. Consultar la ejecución y el registro de la tabla; comprobar el estado del proveedor y las credenciales desde n8n sin revelar claves.
3. Corregir la causa en un entorno de prueba y repetir con un documento sintético; comprobar el registro antes de reanudar.

## Métricas pendientes

El prototipo guarda identificador de ejecución y resultado; aún no implementa UUID de transacción, tokens ni costo por agente. No inferir costos o éxito a partir de datos no registrados.

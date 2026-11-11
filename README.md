# Prototipo de clasificación de soporte

Prototipo académico local con datos sintéticos. El formulario recibe un PDF, Gemini extrae datos, un nodo valida el JSON, un segundo nodo propone una ruta y una tabla local registra extracción, propuesta, fallo del decisor y ruta de revisión. No está desplegado en producción.

## Configuración

Importar flujo.json en n8n; crear una credencial propia de Gemini en n8n y asignarla a ambos nodos; crear una tabla de datos con columna de texto resultado y seleccionar su ID en Insert row. Nunca subir claves. El formulario de prueba se inicia desde n8n y no está publicado en internet.

## Evidencia y límites

Las capturas documentan el prototipo local. Las pruebas usaron PDF sintéticos: SOP-001, SOP-002 y SOP-003, además de un archivo corrupto y una falla simulada del segundo modelo. La ejecución 15 dejó categorías vacías y se derivó a revisión; una ejecución de SOP-001 asignó prioridad no presente en el documento. El control conservador mantiene los casos en revisión. Falta validar fechas, evitar inferencias no respaldadas, instrumentar métricas y alertas, probar falla de almacenamiento y preparar un despliegue estable. Ningún resultado constituye aprobación académica.

## Contrato de entrada y resultado

- Entrada probada: formulario local de prueba de n8n; campo binario `documento` con PDF sintético. No hay endpoint público ni servicio permanente configurado.
- Extracción solicitada: `ticket_id`, `fecha` en AAAA-MM-DD, `categorias` como lista, `prioridad` y `revision_humana`. El modelo puede fallar o inventar valores: el formato solicitado no garantiza exactitud.
- Destino probado: una fila de la tabla local de n8n, con resultado y metadatos de control; consultar las capturas y el JSON exportado para el mapeo concreto.

## Matriz de pruebas observadas

| Prueba local | Observación |
| --- | --- |
| SOP-001, PDF sintético | ID y dos categorías extraídos; revisión humana. En otra ejecución se observó prioridad inventada. |
| SOP-002, PDF sintético | Categorías vacías; revisión humana. |
| SOP-003, PDF sintético | Se registraron categorías de facturación. |
| PDF corrupto | Ejecución 12: se registró revisión humana. |
| Fallo simulado del segundo modelo | Ejecución 14: se registró `fallo_decisor=true` y `ruta_control=revision_humana`; luego se restauró el modelo. |

Las pruebas son locales y no demuestran precisión general, notificaciones, agentes autónomos ni preparación para producción. El flujo incluye dos llamadas a modelos con roles distintos; no implementa agentes autónomos que usan herramientas. El acceso web de entrada, normalización robusta de fechas, costes por ejecución, alertas y prueba de fallo del almacenamiento siguen pendientes.

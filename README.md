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

## Ejemplo de resultado observado

Registro de la ejecución local 8 con el documento sintético SOP-001 (se omite el texto de la propuesta en este extracto):

```json
{
  "extraccion": {
    "ticket_id": "SOP-001",
    "fecha": "2026-11-11",
    "categorias": ["Facturación", "Acceso a la cuenta"],
    "prioridad": null,
    "revision_humana": true,
    "estado": "revision"
  },
  "ejecucion": "8"
}
```

El JSON exportado se encuentra también en `workflow/flujo.json`; las capturas revisadas están en `assets/`. El ejemplo es un extracto real del registro local, no una prueba de webhook ni de normalización.


## Actualización del 11/11/2026: webhook y normalización

El export actualizado está en `workflow/flujo.json` y en la raíz. Se añadieron un webhook POST local, un selector para el binario `documento`/`documento0` y normalización de fechas e importes. El webhook de prueba recibió un PDF sintético; la ejecución completa 21 finalizó correctamente, y la ejecución 22 del webhook publicado localmente finalizó correctamente. La respuesta HTTP 200 indica aceptación, no garantiza por sí sola que los nodos posteriores hayan terminado; el historial confirmó el resultado de la ejecución 22.

La normalización se verificó con cuatro pruebas locales de código (fecha textual, fecha numérica, fecha inválida e importe ambiguo) y con la salida de Gemini en n8n. Todo resultado se envía a revisión humana porque se observó que el modelo puede inventar prioridad. Estos casos no demuestran precisión general. El webhook corre únicamente en el servidor local y no cuenta con autenticación; no usarlo con datos reales ni exponerlo a internet.

Para importar el export sanitizado, crear la credencial Gemini en n8n, reasignar ambos nodos, crear la tabla y seleccionarla en Insert row. El path del webhook fue reemplazado por un marcador para que cada entorno lo configure; activar y probar en ese entorno. El export se distribuye inactivo y sin datos fijados ni credenciales.

Pendiente: sanitización efectiva de PDF antes de enviarlo a IA, UUID por transacción, métricas de tokens/costo por agente, alertas, prueba de errores y almacenamiento bajo la nueva versión, y despliegue seguro. El sistema es un prototipo; las dos llamadas al modelo no acreditan agentes autónomos ni preparación productiva. La captura anterior corresponde a la versión con formulario.


### Prueba completa con normalización

La ejecución local 23 procesó el documento sintético SOP-004 mediante el webhook publicado: la fecha `20 de enero de 2024` se transformó en `2024-01-20`, el importe `$ 1.234,56` en `1234.56` y el resultado quedó en revisión humana. Salida observada del nodo de normalización:

```json
{"ticket_id":"SOP-004","fecha":"2024-01-20","categorias":["facturacion"],"prioridad":"media","importe":1234.56,"revision_humana":true,"estado":"revision"}
```

La llamada produjo HTTP 200 y el historial de n8n mostró éxito; el servidor y la tabla son locales. Se mantienen pendientes la sanitización del documento previo al envío a IA, la instrumentación de tokens y costos y las demás medidas de producción.


### Error controlado de la versión actual

El webhook local recibió un PDF sintético corrupto en la ejecución 24. El historial mostró ejecución completa y el nodo de normalización produjo `estado: revision`, `revision_humana: true` y `error: Respuesta invalida`. El HTTP 200 es solo confirmación de recepción; la ejecución y su salida se verificaron por separado en n8n.


## Correspondencia con el esquema del módulo 1

El plan inicial definió `id`, `categoria`, `prioridad`, `evidencia` y `revision`. La implementación actual los relaciona así; las diferencias son pendientes, no conversiones ya implementadas:

| Origen del documento | Esquema inicial | Salida actual | Diferencia o control |
| --- | --- | --- | --- |
| Identificador del ticket | `id` | `ticket_id` | Cambio de nombre. |
| Motivos o temas | `categoria` | `categorias` | El plan prevé un valor enumerado; el flujo devuelve una lista y todavía no aplica el mismo vocabulario. |
| Prioridad textual | `prioridad` | `prioridad` | El plan usa alta/normal/revisar; el modelo puede devolver otros valores. Requiere revisión humana. |
| Pasaje que respalda la clasificación | `evidencia` | No implementado | Falta extraer y comprobar una referencia al documento. |
| Necesidad de revisión | `revision` | `revision_humana` | Actualmente siempre verdadero por control conservador. |
| Fecha del documento | No contemplado | `fecha` | Campo añadido en el prototipo y normalizado como AAAA-MM-DD si es válido. |
| Monto del documento | No contemplado | `importe` | Campo añadido; número normalizado o null si falta/no es reconocible. |

La validación por rama IF solicitada por Ticher está en preparación. El archivo publicado representa la versión ejecutada anterior; no se presenta la propuesta como implementación probada.

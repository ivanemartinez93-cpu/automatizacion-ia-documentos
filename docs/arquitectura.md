# Arquitectura del prototipo local

Webhook POST/formulario → análisis Gemini → normalización y validación → IF.

- Si `valido=true`: propuesta de ruta Gemini → persistencia.
- Si `valido=false`: persistencia directa del caso para revisión.
- El control `revision_humana=true` es obligatorio en ambas rutas; que el dato cumpla el formato no verifica su fidelidad.
- El export se obtuvo con la CLI oficial de n8n. Las credenciales y el ID de la tabla se retiran de la copia pública y se reasignan al importar.

## Evidencia

Ejecución 25: Gemini limitó solicitudes, IF falso y registro de revisión. Ejecución 26: prueba aislada de n8n con respuestas sintéticas, ambas ramas correctas. Ejecución 27: extracción real de PDF sintético, IF verdadero y registro del fallo del decisor por límite del proveedor. El estado de ejecución final significa que el flujo terminó con manejo de errores; no acredita que ambos modelos respondieran correctamente.

## Runbook de incidentes

1. Suspender decisiones automáticas; mantener el registro para revisión humana.
2. Buscar la ejecución y comprobar `fallo_decisor`, `ruta_control` y el error sin copiar claves ni el contenido privado del documento.
3. Si hay límite de solicitudes, revisar la cuota del proveedor y esperar el intervalo indicado. No repetir envíos en bucle ni cambiar a servicios pagos sin autorización.
4. Si hay un documento inválido, corregir el origen y repetir con un PDF de prueba. Si falla almacenamiento, no confirmar persistencia hasta verificar el registro.
5. Después de corregir, ejecutar un caso válido y uno inválido y comprobar rama, destino y estado.

## Configuración y límites pendientes

El webhook publicado es local y carece de autenticación; usar solo datos ficticios. Aún faltan sanitización de PDF antes de la primera API, UUID común y métricas por agente, costos y alertas. No es un despliegue productivo ni una implementación validada de agentes autónomos.

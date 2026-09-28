# Test Log — SnackCheck

| ID | Tipo | Entrada | Resultado esperado | Resultado obtenido | Estado |
|----|------|---------|---------------------|---------------------|--------|
| TC-001 | Funcional (docs) | Body vacío `{}` | JSON autoexplicativo, 200 | JSON con `message` y `how_to_use`, 200 | ✅ Pass |
| TC-002 | Error (validación) | `{ "barcode": "abc123" }` | 400 con ejemplo de formato | `{"error":"El código de barras no es válido."...}`, 400 | ✅ Pass |
| TC-003 | Error (integración) | `{ "barcode": "0000000000000" }` (no existe) | 404 con ejemplo de barcode válido | `{"error":"Producto no encontrado."...}`, 404 | ✅ Pass |
| TC-004 | Funcional (no saludable) | `{ "barcode": "3017620422003" }` (Nutella, Nutri-Score E) | Veredicto con tono cauteloso | "Nutella: El placer dulce, pero con cautela..." | ✅ Pass |
| TC-005 | Funcional (trampa Nutri-Score) | `{ "barcode": "5449000000996" }` (Coca-Cola) | Veredicto no saludable pese a semáforo | "Coca-Cola: No es la mejor opción para tu salud..." | ✅ Pass |
| TC-006 | Funcional (saludable) | `{ "barcode": "6111035000430" }` (Sidi Ali, Nutri-Score A) | Veredicto con tono alentador | "¡Sidi Ali, tu snack saludable favorito!..." | ✅ Pass |
| TC-007 | Funcional (datos insuficientes) | `{ "barcode": "6111246651261" }` (producto sin nutrientes catalogados) | 200, respuesta de datos insuficientes | `{"status":"insufficient_data",...}`, 200 | ✅ Pass |
| TC-008 | Rendimiento | 3 solicitudes seguidas a `{ "barcode": "3017620422003" }` | Tiempo de respuesta razonable para un flujo con llamada externa + IA | 3.47s (primera, arranque en frío) / 1.08s / 0.84s | ✅ Pass |

## Notas
- Todas las pruebas se ejecutaron contra la Production URL real del webhook (`POST /nutrition-check`), con datos reales de Open Food Facts y veredictos generados en vivo por el LLM de Groq.
- TC-005 confirma la regla del founder ("la trampa del refresco"): aunque los nutrientes por 100ml no disparen el semáforo, el Nutri-Score bajo hace que el veredicto final sea "no saludable".
- **Bug encontrado y corregido durante TC-007**: en el nodo "Respond to Webhook3", el campo `product_name` del JSON de respuesta salía como texto literal `"{{ $json.product.product_name }}"` en vez de evaluarse. Causa: el campo "Response Body" no estaba en modo expresión. Se corrigió activándolo, y se confirmó con una nueva prueba que el campo se evalúa correctamente (queda vacío para este producto en concreto porque Open Food Facts no tiene `product_name` catalogado para ese barcode).
- TC-008: la primera solicitud tarda más (~3.5s) porque el modelo de Groq y la conexión del webhook arrancan en frío; las siguientes bajan a ~1s o menos, tiempo razonable dado que el flujo hace una llamada externa (Open Food Facts) y una llamada a un LLM antes de responder.
- **Revisión post-entrega**: se añadió un nodo Merge (Append) para combinar dos lecturas independientes (semáforo y Nutri-Score) calculadas en paralelo, más naming de nodos, notas inline, el campo de energía (`energy-kcal_100g` con notación de corchetes), y marcadores rojo/verde/amarillo en la respuesta final. Se repitieron TC-004, TC-005 y TC-006 tras el cambio de arquitectura y los tres siguen dando el resultado correcto, confirmando que la nueva estructura con Merge no rompió la lógica de clasificación.

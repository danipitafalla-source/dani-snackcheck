# Changelog — SnackCheck

## [1.1.0] - 2026-09-28
### Added
- Nodo **Merge (Append)**: el semáforo y el Nutri-Score ahora se calculan como dos lecturas independientes en paralelo (tal como describe el brief) y se combinan con Merge antes de determinar el nivel final.
- Campo `energy_kcal` (lectura de `nutriments['energy-kcal_100g']` con notación de corchetes), mostrado en la respuesta final.
- Marcadores 🔴/🟡/🟢 para azúcar, sal y grasa en el veredicto final, además del titular y párrafo generado por la IA.
- Nombres de nodo con formato `[Acción] - [Propósito]` y notas inline en todos los nodos del flujo.

### Changed
- El cálculo del nivel general pasó de un único nodo Set a: dos nodos Edit Fields en paralelo (semáforo, Nutri-Score) → Merge → un nodo Code que consolida ambas lecturas en un solo ítem y calcula el nivel final, evitando duplicar la llamada al LLM.

## [1.0.0] - 2026-09-28
### Added
- Primera versión completa y probada del flujo, de principio a fin.
- Webhook `POST /nutrition-check` con validación de entrada (vacío, no numérico).
- Integración con Open Food Facts para obtener datos reales de producto.
- Manejo de producto no encontrado (404) y datos nutricionales insuficientes (200).
- Clasificación por semáforo (azúcar, sal, grasa) y por Nutri-Score, con regla del "peor de los dos".
- Cálculo de puntuación de preocupación (0-3).
- Generación de veredicto con LLM de Groq, con tono adaptado al nivel de salud.
- Respuesta final como texto plano.

### Fixed
- El campo `product_name` en la respuesta de "datos insuficientes" salía como texto literal sin evaluar (`{{ $json.product.product_name }}`) en vez de mostrar el valor real. Corregido activando el modo expresión en el campo "Response Body" del nodo `Respond to Webhook3`.

## [0.1.0] - 2026-09-28
### Added
- Diagrama inicial en Excalidraw con todo el flujo planificado (decisiones, rutas de error, paso de IA y respuesta final).
- Estructura base del workflow: Webhook, validaciones y llamada a la API.

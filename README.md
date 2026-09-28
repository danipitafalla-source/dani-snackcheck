# SnackCheck — Verificador de Salud Nutricional

## Purpose
Automatización en n8n que recibe el código de barras de un producto envasado y devuelve un veredicto corto, en lenguaje humano, sobre si es una opción saludable, moderada o poco saludable para el día a día. Los datos nutricionales se obtienen en tiempo real de Open Food Facts.

## How It Works
1. Un webhook (`POST /nutrition-check`) recibe `{ "barcode": "..." }`.
2. Se valida que el barcode no esté vacío y que sea numérico.
3. Se consulta Open Food Facts (`GET /api/v2/product/{barcode}.json`) para obtener nombre, marca, Nutri-Score y nutrientes.
4. Se comprueba que el producto exista y que tenga datos nutricionales suficientes.
5. Se calculan dos lecturas independientes en paralelo: el semáforo (azúcar, sal, grasa, energía) y el nivel de Nutri-Score.
6. Un nodo **Merge (Append)** combina ambas lecturas en un único flujo de ítems.
7. Un nodo Code consolida los dos ítems en uno solo y determina el nivel final como el peor de las dos lecturas.
8. Se calcula una puntuación de preocupación (0-3, nº de nutrientes altos).
9. Un LLM de Groq redacta un veredicto corto, con tono alentador o cauteloso según el nivel.
10. Se responde como texto plano, con el veredicto de la IA más un bloque de datos clave (azúcar/sal/grasa con marcador 🔴/🟡/🟢, y energía en kcal).

## Setup
1. Importar `workflow.json` en un workspace de n8n.
2. Configurar una credencial de Groq (API key propia) en el nodo "Groq Chat Model".
3. Activar el workflow para obtener la Production URL del webhook.

## Usage
```
POST /nutrition-check
Content-Type: application/json

{ "barcode": "3017620422003" }
```
Respuesta: texto plano con el veredicto.

## Error Handling
- Barcode vacío/ausente → 200 con instrucciones de uso.
- Barcode no numérico → 400 con ejemplo de formato correcto.
- Producto no encontrado en Open Food Facts → 404 con ejemplo de barcode válido.
- Producto sin datos nutricionales suficientes → 200 indicando que no hay datos.

## Limitations
- Depende de que Open Food Facts tenga el producto catalogado con datos de nutrientes completos.
- Procesa un único producto por solicitud (sin soporte de lotes).
- El tono del veredicto depende del modelo de IA usado (Groq), no es determinista palabra por palabra.

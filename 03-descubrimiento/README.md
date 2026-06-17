# Contexto 3 — Descubrimiento y personalización

Qué se muestra y en qué orden: búsqueda, feed, relacionados, tendencias.
El "video" aquí es un **candidato de ranking** (referencia ligera).

- `openapi.yaml` — especificación de la API (incluye la descripción de responsabilidad)
- `diagrama-clases.drawio` — modelo conceptual
- `diagrama-secuencia.drawio` — flujo "indexar por evento + servir feed"

Eventos que emite: BusquedaRealizada, RecomendacionServida (opcional).
Eventos que consume: ContenidoPublicado/Actualizado/Despublicado, RestriccionAplicada (Catálogo);
SenalEngagement (Audiencia); SesionReproduccionCompletada (Publicación).

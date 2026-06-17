# Contexto 2 — Catálogo editorial y derechos

Qué contenido existe públicamente y bajo qué reglas. El "video" aquí es un **ítem editorial**.

- `openapi.yaml` — especificación de la API (incluye la descripción de responsabilidad)
- `diagrama-clases.drawio` — modelo conceptual
- `diagrama-secuencia.drawio` — flujo "del asset listo a la publicación"

Eventos que emite: ContenidoPublicado, ContenidoDespublicado, ContenidoActualizado,
RestriccionAplicada/Levantada, ReclamacionCreada/Resuelta.
Eventos que consume: AssetListoParaPublicacion (Publicación).

# Contexto 4 — Audiencia, comunidad y engagement

Gestiona la **relación social** entre usuarios, creadores y contenido.
El "canal" aquí es un **perfil social** (no una entidad editorial);
la "suscripción" es seguimiento, no pago (eso es Monetización).

- `openapi.yaml` — especificación de la API (incluye la descripción de responsabilidad)
- `diagrama-clases.drawio` — modelo conceptual
- `diagrama-secuencia.drawio` — flujo "espectador interactúa con un video"

Eventos que emite: UsuarioSuscrito, UsuarioDesuscrito,
ComentarioPublicado, ComentarioEliminado, ReaccionRegistrada, SenalEngagement.

Eventos que consume: ContenidoPublicado (Catálogo) — para notificar a suscriptores;
ContenidoDespublicado / RestriccionAplicada (Catálogo) — para ocultar interacciones.

# Contexto 5 — Monetización

Cómo el creador genera y cobra ingresos en la plataforma. El **"canal"** aquí
es una **entidad económica** (saldo, revenue share, elegibilidad), no un perfil
social ni una entidad editorial.

- `openapi.yaml` — especificación de la API (incluye la descripción de responsabilidad)
- `diagrama-clases.drawio` — modelo conceptual (9 entidades del dominio económico)
- `diagrama-secuencia.drawio` — flujo "ingreso publicitario → atribución → retiro"

---

## Responsabilidad

| Qué sí cubre | Qué NO cubre |
|---|---|
| Elegibilidad para el Programa de Socios (RF-M1) | Facturación al anunciante → **Publicidad** |
| Habilitación/deshabilitación de monetización por canal y contenido (RF-M2) | Ranking / recomendación → **Descubrimiento** |
| Registro y consulta de ingresos del creador — publicidad, membresías, propinas (RF-M3) | Interacciones sociales (likes, comentarios, suscripciones de seguimiento) → **Audiencia** |
| Membresías de pago a canales y propinas/super-gracias (RF-M4) | Procesamiento técnico del video → **Publicación** |
| Retiros: métodos de cobro y solicitudes de pago al creador (RF-M5) | Metadata editorial del contenido → **Catálogo** |

---

## Cobertura de requisitos

| RF | Descripción | Operaciones cubiertas |
|---|---|---|
| RF-M1 | Evaluación de elegibilidad para el Programa de Socios | `GET /canales/{id}/elegibilidad`, `POST /canales/{id}/elegibilidad/evaluar`, `GET|PUT /canales/{id}/monetizacion` |
| RF-M2 | Habilitación/deshabilitación por canal y por contenido | `GET|PUT /canales/{id}/monetizacion`, `GET|PUT /canales/{id}/contenidos/{id}/monetizacion` |
| RF-M3 | Registro y consulta de ingresos del creador | `GET /canales/{id}/ingresos`, `GET /canales/{id}/ingresos/resumen`, `GET /canales/{id}/saldo` |
| RF-M4 | Membresías de pago y propinas (super-gracias) | `GET|POST|PUT|DELETE /canales/{id}/membresias[/{id}]`, `PUT|DELETE /canales/{id}/membresias/{id}/suscribirse`, `GET /usuarios/{id}/membresias`, `POST /contenidos/{id}/propinas`, `GET /contenidos/{id}/propinas/totales` |
| RF-M5 | Métodos de cobro y solicitudes de retiro | `GET|POST /canales/{id}/metodos-pago`, `PATCH|DELETE /canales/{id}/metodos-pago/{id}`, `GET|POST /canales/{id}/retiros`, `GET /canales/{id}/retiros/{id}` |

---

## Eventos

**Emite** (nombres canónicos según `docs/registro-eventos.md`):

| Evento | Disparador |
|---|---|
| `MonetizacionHabilitada` | Creador o sistema habilita monetización (canal o contenido) |
| `MonetizacionDeshabilitada` | Creador deshabilita monetización, o llega `RestriccionAplicada` tipo `monetizacion` desde Catálogo |
| `IngresoAtribuido` | Se registra un ingreso neto al creador (publicidad, membresía o propina) |
| `PagoProcesado` | La pasarela de pagos confirma la transferencia de un retiro |

**Consume**:

| Evento | Fuente | Reacción |
|---|---|---|
| `ContenidoPublicado` | Catálogo | Registra el contenido como candidato a monetización |
| `ContenidoDespublicado` | Catálogo | Suspende atribución de ingresos pendientes; mantiene los ya confirmados |
| `RestriccionAplicada` (tipo `monetizacion`) | Catálogo | Deshabilita monetización del contenido y emite `MonetizacionDeshabilitada` |
| `IngresoPublicitarioGenerado` | Publicidad | Aplica revenue share y crea un `Ingreso` de fuente `publicidad` |

---

## Decisiones de diseño (ver `docs/registro-eventos.md`)

| Decisión | Resolución |
|---|---|
| **1 — Elegibilidad (RF-M1)** | Consulta síncrona a Audiencia (suscriptores) y Publicación (horas de visualización). Monetización guarda solo el resultado y la fecha, no los conteos en tiempo real. |
| **2 — Visibilidad hacia Publicidad** | `MonetizacionDeshabilitada` es el canal único que Publicidad consume para dejar de servir anuncios. Llega tanto por acción del creador como por `RestriccionAplicada` de Catálogo. |
| **4 — Membresías y propinas** | Se procesan como operaciones internas de este contexto. Sin séptimo contexto de pagos. Los datos bancarios sensibles los tokeniza la pasarela externa; aquí solo vive el token opaco. |

---

## Consistencia eventual (RNF-5)

Caso límite documentado: video despublicado en Catálogo con ingresos publicitarios en vuelo.

- **Ingresos ya confirmados** por `IngresoPublicitarioGenerado` → se mantienen (el anunciante ya fue facturado).
- **Ingresos pendientes** de confirmación → se cancelan al procesar `ContenidoDespublicado`.
- Durante la ventana de procesamiento, `GET /ingresos/resumen` devuelve los ingresos con `advertencia: ingresos-parciales` y `estadoContenido: pendiente-verificacion`.

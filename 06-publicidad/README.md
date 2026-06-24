# Contexto 6 — Publicidad

Gestiona el **ecosistema publicitario** de la plataforma. El **"video"** aquí
es un **InventorySlot** (espacio publicitario): un slot donde puede mostrarse
un anuncio durante una sesión de reproducción. El "canal" es una **fuente de
inventory**, no un perfil social ni una entidad económica.

- `openapi.yaml` — especificación de la API (incluye la descripción de responsabilidad)
- `diagrama-clases.drawio` — modelo conceptual (9 entidades del dominio publicitario)
- `diagrama-secuencia.drawio` — flujo "del contenido publicado a la impresión facturable"

---

## Responsabilidad

| Qué sí cubre | Qué NO cubre |
|---|---|
| Alta y gestión de cuentas de anunciante (RF-F1) | Payouts al creador → **Monetización** |
| Creación de campañas, creativos y control de presupuesto (RF-F1, RF-F2) | Ranking / recomendación de contenido → **Descubrimiento** |
| Targeting por categoría, país y rango etario (RF-F3) | Interacciones sociales (likes, comentarios) → **Audiencia** |
| Brand safety: exclusión de categorías y palabras clave (RF-F4) | Metadata editorial del video → **Catálogo** |
| Gestión del inventory de slots publicitarios (RF-F5) | Procesamiento técnico del video → **Publicación** |
| Decisión de anuncio en una oportunidad de visualización — ad decision (RF-F6) | |
| Registro de impresiones facturables y facturación al anunciante (RF-F7) | |
| Emisión de ingreso bruto hacia Monetización (RF-F7) | |

---

## Cobertura de requisitos

| RF | Descripción | Operaciones cubiertas |
|---|---|---|
| RF-F1 | Gestión de anunciantes, campañas y creativos | `POST|GET|PATCH /anunciantes[/{id}]`, `GET|POST /anunciantes/{id}/campanas`, `GET|PATCH /campanas/{id}`, `GET|POST /campanas/{id}/creativos[/{id}]` |
| RF-F2 | Control de presupuesto y estado de campaña | `GET /campanas/{id}` (presupuestoRestante en tiempo real), `PATCH /campanas/{id}` (pausar/reactivar), evento `PresupuestoAgotado` |
| RF-F3 | Targeting por audiencia, categoría y territorio | `GET|PUT /campanas/{id}/targeting` |
| RF-F4 | Brand safety — exclusión de contenido inadecuado | `GET|PUT /campanas/{id}/brand-safety` |
| RF-F5 | Inventory: slots publicitarios de contenido monetizable | `GET /inventory`, `GET /inventory/{catalogItemId}` (consulta síncrona RNF-4) |
| RF-F6 | Ad decision en una oportunidad de visualización | `POST /ad-decisions`, `GET /ad-decisions/{id}` |
| RF-F7 | Impresiones facturables, facturación y emisión de ingreso | `POST /impresiones`, `GET /campanas/{id}/impresiones`, `GET|GET /campanas/{id}/facturacion[/{periodo}/reporte]` |

---

## Eventos

**Emite** (nombres canónicos según `docs/registro-eventos.md`):

| Evento | Disparador |
|---|---|
| `AnuncioServido` | Ad decision ejecutada — creativo ganador seleccionado (RF-F6) |
| `ImpresionFacturableConfirmada` | Impresión validada para facturación al anunciante (RF-F7) |
| `IngresoPublicitarioGenerado` | Impresión confirmada — genera el ingreso bruto que Monetización distribuye (RF-F7) |
| `PresupuestoAgotado` | Presupuesto de campaña llega a cero tras descontar una impresión (RF-F2) |

**Consume**:

| Evento | Fuente | Reacción |
|---|---|---|
| `ContenidoPublicado` | Catálogo | Crea un `InventorySlot` con categoría y flag monetizable del payload (decisión 5) |
| `ContenidoDespublicado` | Catálogo | Cambia el slot a estado `suspendido`; no se sirven más anuncios |
| `RestriccionAplicada` | Catálogo | Aplica restricción de brand safety; puede pasar el slot a `restringido` según tipo |
| `MonetizacionDeshabilitada` | Monetización | Cambia el slot a `suspendido` y `monetizable: false` (decisión 2) |
| `SenalEngagement` | Audiencia | Actualiza señales de afinidad de audiencia usadas por el motor de targeting (decisión 5) |

---

## Decisiones de diseño (ver `docs/registro-eventos.md`)

| Decisión | Resolución |
|---|---|
| **2 — Señal de monetización hacia Publicidad** | Publicidad consume `MonetizacionDeshabilitada` de Monetización como canal primario para suspender anuncios, complementando `RestriccionAplicada` tipo `monetizacion` de Catálogo. Ambas producen la misma reacción: slot a `suspendido`. |
| **5 — Fuente de categoría para brand safety** | La categoría del contenido llega en el payload de `ContenidoPublicado` (Catálogo, campo `categoria`). Publicidad la almacena localmente en el `InventorySlot`. No hay consulta síncrona a Catálogo ni a Audiencia en tiempo de decisión. |

---

## Consistencia eventual (RNF-5)

Caso límite documentado: campaña cuyo presupuesto se agota mientras hay `AdDecision`
en estado `servido` cuya impresión aún no fue confirmada.

- Al procesar la última impresión que agota el presupuesto, se emite `PresupuestoAgotado` y la campaña pasa a `agotada`.
- Todas las `AdDecision` en estado `servido` pasan automáticamente a `expirada`.
- Si llega `POST /impresiones` referenciando una `AdDecision` expirada, la API responde **409** con `{ codigo: PRESUPUESTO_AGOTADO }`.
- El anunciante **nunca es facturado** por impresiones posteriores al agotamiento.
- `IngresoPublicitarioGenerado` solo se emite para impresiones confirmadas antes del agotamiento.

# Registro unificado de eventos entre bounded contexts

> **Propósito:** fuente única de verdad para los nombres y contenidos de los eventos que conectan los 6 contextos. Cada especificación OpenAPI debe referenciar los eventos usando **exactamente estos nombres** (en la sección `description` o en `webhooks`). Así garantizamos que lo que un contexto emite coincide con lo que otro consume (criterio de evaluación 3: integración sin acoplamiento indebido).

---

## Convenciones

- **Nombre del evento:** español, PascalCase, verbo en pasado cuando aplica (ej. `ContenidoPublicado`).
- **IDs compartidos:** se usan tal como los nombra el enunciado en RNF-2: `videoAssetId`, `catalogItemId`, `channelId`, `userId`. Compartimos identificadores, **nunca** el modelo completo (RNF-2).
- **Payload mínimo:** cada evento lleva solo los datos imprescindibles para que el consumidor reaccione. Si un consumidor necesita más información, hace una **consulta síncrona** al contexto dueño del dato (RNF-4).
- **Sobre común (envelope):** todo evento viaja con esta estructura:

```yaml
eventId: string        # identificador único del evento
evento: string         # nombre canónico (de este registro)
emisor: string         # contexto que lo emite
fechaHora: date-time   # cuándo ocurrió
datos: { ... }         # payload específico (ver tablas)
```

---

## Tabla maestra de eventos

### Emitidos por Publicación (contexto 1)

| Evento | Consumidores | Disparador (RF) | Payload mínimo (`datos`) |
|---|---|---|---|
| `ContenidoSubido` | — | Subida validada completa (RF-P1) | videoAssetId, userId |
| `ProcesamientoFinalizado` | — | Fin del transcoding, con éxito o fallo (RF-P2) | videoAssetId, resultado (exito\|fallo), calidades[] |
| `AssetListoParaPublicacion` | **Catálogo** | Procesamiento exitoso; el asset puede vincularse a metadata editorial (RF-P2) | videoAssetId, userId, duracionSeg, calidades[] |
| `SesionReproduccionCompletada` | **Descubrimiento** | Fin de una sesión de playback (RF-P3) | sessionId, videoAssetId, userId (opcional), duracionVistaSeg, calidadUsada |
| `TransmisionIniciada` / `TransmisionFinalizada` | — | Inicio / fin de un live (RF-P4) | liveId, userId; al finalizar agrega videoAssetId del VOD generado |

### Emitidos por Catálogo (contexto 2)

| Evento | Consumidores | Disparador (RF) | Payload mínimo (`datos`) |
|---|---|---|---|
| `ContenidoPublicado` | **Descubrimiento, Audiencia, Monetización, Publicidad** | El creador publica un contenido (RF-C3) | catalogItemId, videoAssetId, channelId, categoria, restricciones resumidas (edadMinima, paisesBloqueados, monetizable) |
| `ContenidoDespublicado` | **Descubrimiento, Audiencia, Monetización, Publicidad** | Despublicar u ocultar (RF-C3) | catalogItemId |
| `ContenidoActualizado` | **Descubrimiento** | Edición de metadata editorial (RF-C2) | catalogItemId, camposCambiados[] |
| `RestriccionAplicada` / `RestriccionLevantada` | **Descubrimiento, Publicidad, Monetización, Audiencia** | Política o reclamación que cambia el estado (RF-C5, RF-C6) | catalogItemId, tipoRestriccion (edad\|geo\|monetizacion\|moderacion) |
| `ReclamacionCreada` / `ReclamacionResuelta` | — | Registro o resolución de reclamación de derechos (RF-C5) | claimId, catalogItemId, estado |

### Emitidos por Descubrimiento (contexto 3)

| Evento | Consumidores | Disparador (RF) | Payload mínimo (`datos`) |
|---|---|---|---|
| `BusquedaRealizada` | — | Usuario ejecuta una búsqueda (RF-D1) | query, userId (opcional), filtros |
| `RecomendacionServida` *(opcional según enunciado)* | — | Se entrega un feed o lista de relacionados (RF-D2, RF-D3) | userId, catalogItemIds[] |

### Emitidos por Audiencia (contexto 4)

| Evento | Consumidores | Disparador (RF) | Payload mínimo (`datos`) |
|---|---|---|---|
| `UsuarioSuscrito` / `UsuarioDesuscrito` | — *(ver decisión abierta 1)* | Seguir / dejar de seguir un canal (RF-A1) | userId, channelId |
| `ComentarioPublicado` / `ComentarioEliminado` | — | Acción sobre comentarios (RF-A3) | commentId, catalogItemId, userId |
| `ReaccionRegistrada` | — | Like, dislike u otra reacción (RF-A2) | catalogItemId, userId, tipo |
| `SenalEngagement` | **Descubrimiento, Publicidad** | Agregación de interacciones del contenido/canal (RF-A1, RF-A2, RF-A3) | catalogItemId, channelId, tipoSenal, valor, ventanaTiempo |

### Emitidos por Monetización (contexto 5)

| Evento | Consumidores | Disparador (RF) | Payload mínimo (`datos`) |
|---|---|---|---|
| `MonetizacionHabilitada` / `MonetizacionDeshabilitada` | — *(ver decisión abierta 2)* | Cambio de estado de monetización a nivel canal o contenido (RF-M1) | channelId o catalogItemId, alcance |
| `IngresoAtribuido` | — | Ingreso asignado al creador tras revenue share (RF-M3) | userId (creador), monto, fuente (publicidad\|membresia\|propina), periodo |
| `PagoProcesado` | — | Retiro ejecutado (RF-M5) | payoutId, userId, monto |

### Emitidos por Publicidad (contexto 6)

| Evento | Consumidores | Disparador (RF) | Payload mínimo (`datos`) |
|---|---|---|---|
| `AnuncioServido` | — | Decisión de ad ejecutada en una oportunidad de visualización (RF-F6) | adDecisionId, campaignId, creativeId, catalogItemId |
| `ImpresionFacturableConfirmada` | — | Impresión validada para facturación al anunciante (RF-F7) | impressionId, campaignId, monto |
| `IngresoPublicitarioGenerado` | **Monetización** | Impresión pagada confirmada que genera ingreso a repartir (RF-F7 + restricción de diseño del contexto 6) | revenueEventId, campaignId, catalogItemId, channelId, montoBruto |
| `PresupuestoAgotado` | — | Presupuesto de campaña llega a cero (RF-F2) | campaignId |

> Los eventos con consumidor "—" igual son **obligatorios** (el enunciado exige emitirlos), aunque ningún otro contexto los consuma hoy.

---

## Discrepancias del enunciado que este registro resuelve

El enunciado nombra el mismo evento de formas distintas según la sección. Estos son los nombres canónicos que adoptamos:

| Nombre canónico | Cómo lo llama el enunciado al emitir | Cómo lo llama al consumir |
|---|---|---|
| `AssetListoParaPublicacion` | "Asset listo para publicación editorial" (Publicación, línea 123) | "Asset técnicamente listo" (Catálogo, línea 194) |
| `SenalEngagement` | "Señal de engagement agregada" (Audiencia, línea 330) | "Engagement registrado" (Descubrimiento, línea 256); "Señales de engagement" (Publicidad, línea 470) |
| `IngresoPublicitarioGenerado` | "Ingreso publicitario generado" (Publicidad, línea 476) | "Ingreso publicitario confirmado" (Monetización, línea 392) |
| `ContenidoDespublicado` + `RestriccionAplicada` | "despublicado" / "Restricción aplicada" (Catálogo) | "Cambios de visibilidad de contenido" (Audiencia, línea 335); "restringido" (Descubrimiento / Monetización / Publicidad) |

---

## Decisiones abiertas (resolver en equipo)

1. **Elegibilidad de Monetización (RF-M1):** necesita suscriptores y horas de visualización, pero el enunciado no declara cómo los obtiene. Propuesta: consulta síncrona a Audiencia (suscriptores) y a Descubrimiento o Publicación (watch time). Decidir y documentarlo en la spec de Monetización.
2. **¿Publicidad consume `MonetizacionHabilitada/Deshabilitada`?** RF-F5 dice que Publicidad debe respetar "contenido no monetizable", pero su lista de consumo solo declara Catálogo. Decidir si la señal de monetización viaja vía la restricción de Catálogo (`RestriccionAplicada` tipo `monetizacion`) o vía evento directo de Monetización.
3. **Granularidad de `SenalEngagement`:** el enunciado dice "agregada" — decidir si se emite por interacción individual o agregada por ventana de tiempo.
4. **Membresías y propinas:** el enunciado (línea 394) deja abierto si la compra se origina dentro de Monetización o en "un contexto de pagos compartido". Recomendación: tratarla como operación interna de Monetización para no inventar un séptimo contexto.
5. **Fuente de "categoría" para brand safety (Publicidad):** el enunciado menciona "Audiencia / Descubrimiento" como fuente, pero la categoría es metadata de Catálogo. Propuesta: viaja en el payload de `ContenidoPublicado`.

---

*Última actualización: pendiente de revisión por el equipo en la reunión de kickoff.*

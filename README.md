# Proyecto YouTube — Diseño con Domain-Driven Design

Modelado de YouTube como **6 bounded contexts independientes**, cada uno con su
propia especificación OpenAPI, diagrama de clases y diagrama de secuencia.

Curso: Análisis y Diseño de Software · Entrega: 24 de junio de 2026

---

## Los 6 bounded contexts

| # | Contexto | Responsabilidad | Estado |
|---|----------|-----------------|--------|
| 1 | [Publicación](01-publicacion/) | Ciclo de vida técnico del video: subida, procesamiento, reproducción, transmisión | ✅ |
| 2 | [Catálogo](02-catalogo/) | Qué contenido existe públicamente y bajo qué reglas (editoriales, legales, de visibilidad) | ✅ |
| 3 | [Descubrimiento](03-descubrimiento/) | Qué se muestra y en qué orden: búsqueda, feed, relacionados, tendencias | ✅ |
| 4 | [Audiencia](04-audiencia/) | Relación social: suscripciones, likes, comentarios, notificaciones, historial | ⬜ |
| 5 | [Monetización](05-monetizacion/) | Cómo el creador gana dinero: elegibilidad, ingresos, membresías, retiros | ⬜ |
| 6 | [Publicidad](06-publicidad/) | Campañas de anunciantes, anuncios en videos, facturación | ⬜ |

## Idea central del diseño

El mismo concepto **"video"** se modela de forma **distinta en cada contexto** —
duplicación intencional del modelo, no una entidad global compartida:

- En **Publicación** es un *asset técnico* (códec, calidades, estado de procesamiento)
- En **Catálogo** es un *ítem editorial* (título, descripción, visibilidad)
- En **Descubrimiento** es un *candidato de ranking* (referencia ligera con señales)

Los contextos comparten **identificadores** (`videoAssetId`, `catalogItemId`,
`channelId`, `userId`), nunca modelos completos.

## Cómo se comunican

Preferentemente por **eventos asíncronos** a través de un bus; consultas
síncronas solo para validar existencia o visibilidad. El catálogo completo de
eventos (nombres canónicos y payloads) está en
[`docs/registro-eventos.md`](docs/registro-eventos.md) — **fuente única de
verdad**: cada contexto debe usar esos nombres exactos.

### Mapa de dependencias

```
Publicación ──AssetListoParaPublicacion──▶ Catálogo
Catálogo ──ContenidoPublicado──▶ Descubrimiento, Audiencia, Monetización, Publicidad
Audiencia ──SenalEngagement──▶ Descubrimiento
Publicación ──SesionReproduccionCompletada──▶ Descubrimiento
Publicidad ──IngresoPublicitarioGenerado──▶ Monetización
```

## Escenario integrador

Flujo de prueba que recorre los 6 contextos (un espectador ve un video
recomendado, mira un anuncio, deja un like y el creador genera ingresos):

| Paso | Acción | Contexto responsable |
|------|--------|---------------------|
| 1 | Obtener recomendación | Descubrimiento — `GET /feed` |
| 2 | Validar que el contenido es visible | Catálogo — `GET /contenidos/{id}/visibilidad` |
| 3 | Elegir anuncio para la oportunidad | Publicidad |
| 4 | Reproducir el video | Publicación — `GET /assets/{id}/reproduccion` |
| 5 | Registrar el like | Audiencia |
| 6 | Registrar watch time | Publicación → Descubrimiento (evento) |
| 7 | Atribuir ingreso al creador | Publicidad → Monetización (evento) |

## Estructura del repositorio

```
DDD_Youtube/
├── 01-publicacion/      openapi.yaml + diagramas
├── 02-catalogo/         openapi.yaml + diagramas
├── 03-descubrimiento/   openapi.yaml + diagramas
├── 04-audiencia/        (pendiente)
├── 05-monetizacion/     (pendiente)
├── 06-publicidad/       (pendiente)
└── docs/
    ├── registro-eventos.md       catálogo de eventos entre contextos
    ├── plantilla-openapi.yaml     convenciones del equipo (RNF-1 a RNF-5)
    └── checklist-entrega.md       verificación previa a la entrega
```

## Cómo revisar las APIs

Cada `openapi.yaml` se puede visualizar pegándolo en
[editor.swagger.io](https://editor.swagger.io). Los diagramas `.drawio` se abren
en [app.diagrams.net](https://app.diagrams.net) o la app de escritorio.

## Equipo

- [Benjamin Eduardo Campos Carvajal] — contextos 1, 2, 3
- [Benjamin Eduardo Soto Arriaza] — contextos 4, 5, 6

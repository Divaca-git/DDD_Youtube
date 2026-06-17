# Checklist de entrega — Proyecto YouTube (DDD)

> **Cómo usarlo:** cada integrante corre la sección "Por contexto" sobre sus
> dos contextos antes de commitear. La sección "De equipo" se corre entre los
> tres antes del commit final (recomendado: 23 de junio, un día antes del cierre).

---

## Por contexto (correr 6 veces, una por bounded context)

### Entregables presentes (enunciado, sección Entregables)
- [ ] Descripción de responsabilidad dentro del OpenAPI (`info.description`)
- [ ] Especificación OpenAPI que valida sin errores (pegar en editor.swagger.io)
- [ ] Diagrama de clases (borrador conceptual) — `.drawio` + exportado a PNG
- [ ] Diagrama de secuencia de al menos un flujo con integración con otros contextos — `.drawio` + PNG

### RNF transversales
- [ ] **RNF-1:** todas las operaciones incluyen la línea "Quién puede ejecutarla"; `securitySchemes` definido
- [ ] **RNF-2:** solo se comparten IDs (`videoAssetId`, `catalogItemId`, `channelId`, `userId`); ningún modelo de otro contexto copiado
- [ ] **RNF-3:** listados con paginación (o decisión de no paginar documentada); schema de `Error` único en la API
- [ ] **RNF-4:** eventos asíncronos para cambios de estado; consultas síncronas solo para existencia/visibilidad
- [ ] **RNF-5:** al menos un caso límite de consistencia eventual documentado (qué responde la API y por qué)

### Fronteras — errores típicos del enunciado
- [ ] La API cubre UN solo dominio (no es un controlador con todo)
- [ ] **Catálogo:** cero endpoints o clases de comentarios (interacción social → Audiencia)
- [ ] **Audiencia:** cero lógica de ranking o recomendación (eso es Descubrimiento; Audiencia solo emite señales)
- [ ] **Publicidad:** cero payouts al creador (Publicidad factura al anunciante; Monetización paga al creador)
- [ ] El modelo del "video" de este contexto es PROPIO: comparar sus campos con los de las otras specs — no deben coincidir más allá de los IDs

### Coherencia interna
- [ ] Cada RF del contexto está mapeado a al menos una operación o evento (tabla de cobertura)
- [ ] El diagrama de clases y los `components/schemas` cuentan el mismo modelo
- [ ] Los eventos mencionados en las descriptions usan los nombres EXACTOS de `docs/registro-eventos.md`
- [ ] Las restricciones de diseño del contexto (sección "Restricciones" del enunciado) se respetan

---

## De equipo (antes del commit final)

- [ ] **Matriz de eventos cuadra:** todo evento que un contexto consume es emitido por otro con el mismo nombre canónico y payload compatible
- [ ] **Escenario integrador recorrido en vivo** — los 7 pasos del enunciado asignados a endpoints/eventos reales de NUESTRAS specs:
  1. Obtener recomendación → Descubrimiento
  2. Validar que el contenido es visible → Catálogo (`GET /contenidos/{id}/visibilidad`)
  3. Elegir anuncio → Publicidad
  4. Reproducir video → Publicación
  5. Registrar like → Audiencia
  6. Registrar watch time → Descubrimiento (señales)
  7. Atribuir ingreso al creador → Monetización (desde evento de Publicidad)
- [ ] Los 6 YAML validan sin errores
- [ ] Todos los diagramas exportados a PNG (el profe no debería necesitar abrir Draw.io)
- [ ] README raíz con el mapa de contextos y la tabla del escenario integrador
- [ ] Commits de los TRES integrantes visibles en el historial de GitHub
- [ ] Nada de lo "fuera de alcance" quedó modelado (login con Google, moderación profunda/strikes, analytics de creadores, YouTube Studio)
- [ ] Decisiones abiertas del registro de eventos resueltas y documentadas

---

*Origen de cada sección: Entregables (enunciado §Entregables), RNF (§Requisitos no funcionales),
Fronteras (§Errores típicos que se deben evitar), Escenario (§Escenario integrador),
Criterios (§Criterio de evaluación orientativo).*

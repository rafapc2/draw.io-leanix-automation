# Tarea: Convertir diagrama LeanIX a draw.io formato C4

## Instrucción principal

Tengo acceso al MCP de LeanIX y al MCP de draw.io. Quiero que:

1. Leas el diagrama de LeanIX llamado: **"[NOMBRE DEL DIAGRAMA]"**
2. Obtengas el detalle de todas las aplicaciones que contiene
3. Generes una versión equivalente en draw.io usando el estándar C4, 
   en XML nativo (NO uses Mermaid)

---

## Paso 1 — Encontrar el diagrama en LeanIX

Usa el tool `get_diagrams` paginando de a 2 resultados por página hasta 
encontrar el diagrama cuyo `name` coincida exactamente con 
"[NOMBRE DEL DIAGRAMA]". Guarda su `id`.

Luego usa `get_diagram_by_id` con ese id para obtener el XML completo 
del diagrama.

---

## Paso 2 — Extraer las aplicaciones

Del XML del campo `graphXml` dentro de `state`, extrae todos los elementos 
que tengan:

- `type="factSheet"`
- `factSheetType="Application"`

Recopila los valores de:

- `factSheetId` (UUID de la aplicación)
- `label` (nombre visible)

Elimina duplicados (misma aplicación puede aparecer más de una vez).

---

## Paso 3 — Obtener detalles de cada aplicación

Llama a `get_fact_sheet_details` con los UUIDs extraídos, en lotes de 
máximo 25. Para cada aplicación extrae estos campos exactos:

| Campo LeanIX             | Uso en el diagrama C4         |
|--------------------------|-------------------------------|
| `name` / `displayName`   | Nombre del sistema            |
| `description`            | Descripción (primeros 80 car.)|
| `lifecycle.asString`     | Estado del ciclo de vida      |
| `lxHostingType`          | Tipo de hosting               |
| `category`               | Categoría de negocio          |
| `technicalSuitability`   | Fitness técnico               |

---

## Paso 4 — Clasificar las aplicaciones en capas C4

Agrupa las aplicaciones en las siguientes capas según su función 
(usa el nombre, descripción y tags para inferir la capa correcta):

| Capa                    | Tipos de sistemas típicos                          |
|-------------------------|----------------------------------------------------|
| Canales                 | Portales web, apps móviles, ATMs, CRM, contact center |
| Integración             | API Gateways, ESBs, message brokers, buses         |
| Pagos & Autorización    | Switches de pago, autorizadores, wallets           |
| Core Bancario           | Core banking, TPM, motores de tasas/comisiones, MDM|
| Productos Legacy        | Aplicaciones desktop o en phaseOut de negocio      |
| Analítica & Datos       | Datalakes, datamarts, reportes, ERPs, orquestadores|

Si no estás seguro de la capa, usa la descripción y el campo `category`.

---

## Paso 5 — Identificar sistemas externos

Del XML del diagrama, identifica sistemas que se relacionan con el banco 
pero son externos (redes de pago, wallets, cámaras de compensación, etc.). 
Estos van fuera del boundary principal con color gris.

---

## Paso 6 — Generar el XML draw.io en formato C4

Genera el XML usando estas convenciones **obligatorias**:

### Shapes C4 estándar de draw.io

- Persona/usuario: `shape=mxgraph.c4.person2`
- Sistema interno: `shape=mxgraph.c4.system2`
- Sistema externo: `shape=mxgraph.c4.system2` (color gris)
- Contenedor/boundary: `swimlane` con `startSize=30`

### Colores por estado de lifecycle

| Estado       | fillColor | strokeColor | fontColor |
|--------------|-----------|-------------|-----------|
| `active`     | #1168BD   | #0B4884     | #FFFFFF   |
| `phaseOut`   | #E6821E   | #A85B10     | #FFFFFF   |
| `phaseIn`    | #1168BD   | #0B4884     | #FFFFFF   |
| `endOfLife`  | #AE4132   | #7A2A1E     | #FFFFFF   |
| Externo      | #999999   | #666666     | #FFFFFF   |

### Colores por capa (swimlane)

| Capa                  | fillColor | strokeColor | fontColor |
|-----------------------|-----------|-------------|-----------|
| Canales               | #DAE8FC   | #6C8EBF     | #08427B   |
| Integración           | #E1D5E7   | #9673A6     | #4B0073   |
| Pagos & Autorización  | #FFF2CC   | #D6B656     | #7A5A00   |
| Core Bancario         | #D5E8D4   | #82B366     | #1A4F1A   |
| Productos Legacy      | #F8CECC   | #B85450     | #8B0000   |
| Analítica & Datos     | #E8DEF8   | #7C5FA6     | #3D0066   |
| Sistemas Externos     | #F5F5F5   | #AAAAAA     | #666666   |
| Boundary principal    | #EFF4F9   | #23445D     | #23445D   |

### Conectores

- Integración síncrona: `edgeStyle=orthogonalEdgeStyle;rounded=1`
- Integración async/batch: `edgeStyle=orthogonalEdgeStyle;rounded=1;dashed=1`
- Extrae las relaciones directamente del XML original del diagrama 
  (elementos `mxCell` con `edge="1"`, siguiendo `source` y `target` 
  hacia los `factSheetId` correspondientes)

### Label de cada sistema (formato obligatorio)
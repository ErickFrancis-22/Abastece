# Diccionario de datos — Abastece

Diseño inicial. Las tablas aún no han sido implementadas

## Convenciones

- Los nombres de tablas y campos se escriben en inglés.
- PK significa clave primaria.
- FK significa clave foránea.
- Los identificadores int se generan mediante IDENTITY.
- Los textos utilizan nvarchar, excepto los códigos de país.
- Los importes utilizan decimal, no float.
- Las fechas se guardan en UTC mediante datetime2.
- Los códigos de país utilizan dos letras.
- Las relaciones con historial no se eliminan en cascada.

## 1. Categories

Agrupa productos del catálogo.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| Name | nvarchar(100) | No | Nombre único y no vacío. |
| IsActive | bit | No | Indica si admite nuevas asignaciones. |

Una categoría puede tener muchos productos.
Cada producto pertenece a una categoría.

Desactivar una categoría no desactiva automáticamente sus productos
ni elimina sus relaciones existentes.

## 2. Products

Representa un artículo específico del catálogo.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| CategoryId | int | No | FK hacia Categories.Id. |
| Sku | nvarchar(50) | No | Único y no vacío. |
| Name | nvarchar(150) | No | Nombre no vacío. |
| Description | nvarchar(500) | Sí | Descripción adicional. |
| MinimumStock | int | No | Mayor o igual que cero. |
| TargetStock | int | No | Mayor que MinimumStock. |
| CountryOfOriginCode | char(2) | Sí | País de fabricación, si se conoce. |
| ImageUrl | nvarchar(500) | Sí | Referencia a una imagen. |
| IsActive | bit | No | Habilita el producto para nuevas operaciones. |

El producto no almacena el costo de cada proveedor.

Las existencias, la apertura y los movimientos se modelan por separado.

En esta versión, cada SKU representa un producto con un origen definido,
si se conoce. No se rastrea el país por lote.

## 3. Suppliers

Representa una empresa que suministra productos.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| Name | nvarchar(150) | No | Nombre no vacío. |
| CountryCode | char(2) | No | País donde está ubicado el proveedor. |
| ContactName | nvarchar(150) | Sí | Persona de contacto. |
| Email | nvarchar(254) | Sí | Correo de contacto. |
| Phone | nvarchar(30) | Sí | Teléfono; se trata como texto. |
| IsActive | bit | No | Habilita al proveedor para nuevas compras. |

El nombre del proveedor no se considera un identificador único.

El país del proveedor no determina el país de fabricación del producto.

## 4. ProductSuppliers

Relaciona productos y proveedores, y conserva sus condiciones
actuales de suministro.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ProductId | int | No | FK hacia Products.Id. |
| SupplierId | int | No | FK hacia Suppliers.Id. |
| UnitCost | decimal(18,2) | No | Costo unitario en USD, mayor que cero. |
| LeadTimeDays | int | No | Plazo estimado en días calendario, no negativo. |
| MinimumOrderQuantity | int | No | Entero mayor que cero. |
| UpdatedAt | datetime2 | No | Última actualización de las condiciones, en UTC. |
| IsActive | bit | No | Indica si la condición de suministro está vigente. |

### Restricciones

- La combinación ProductId + SupplierId es única.
- La compra mínima no implica múltiplos de empaque.
- Un plazo de cero representa entrega estimada el mismo día.
- Desactivar una relación conserva su historial.
- Si vuelve a utilizarse la misma combinación, se reactiva el registro
  existente en lugar de crear otro.

### Historial de condiciones

Esta tabla contiene las condiciones actuales de referencia.

Las solicitudes y órdenes conservan sus propias condiciones históricas.
Los cambios relevantes del catálogo también se registrarán en auditoría.

Actualizar UnitCost no modifica solicitudes ni órdenes anteriores.

## 5. Actors

Identifica a las personas y agentes responsables de acciones.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ActorType | nvarchar(20) | No | User o Agent. |
| DisplayName | nvarchar(150) | No | Nombre visible, no vacío. |
| CreatedAt | datetime2 | No | Fecha de creación en UTC. |

Cada actor corresponde exclusivamente a un usuario o a un agente,
según ActorType.

La identidad permite atribuir acciones, pero no concede permisos.

Los actores con historial no se eliminan físicamente.

## 6. Users

Contiene los datos de acceso de las personas.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ActorId | int | No | FK hacia Actors.Id; único. |
| Email | nvarchar(254) | No | Correo de acceso. |
| NormalizedEmail | nvarchar(254) | No | Correo normalizado; único. |
| PasswordHash | nvarchar(max) | No | Hash generado por el mecanismo de autenticación. |
| IsActive | bit | No | Permite o bloquea el acceso. |
| CreatedAt | datetime2 | No | Fecha de creación en UTC. |

### Restricciones

- El actor relacionado debe tener ActorType = User.
- El usuario y su actor se crean en una misma transacción.
- La normalización del correo se realiza mediante una política
  consistente de la aplicación.
- La contraseña original no se almacena ni se registra en logs.
- Desactivar al usuario no elimina sus acciones anteriores.
- Los permisos se obtienen de sus asignaciones de roles vigentes.

## 7. Roles

Define los roles disponibles en la primera versión.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| Name | nvarchar(50) | No | Nombre único. |

Roles iniciales:

- Administrator: usuarios, asignaciones y configuración.
- Manager: aprobación de compras y ajustes; supervisión.
- InventoryOperator: catálogos y operación de inventario.

La interfaz puede mostrar los nombres en español.

Los roles son predefinidos. La primera versión permite asignarlos
y retirarlos, pero no crear roles ni editar sus permisos.

Administrator no incluye automáticamente los otros roles.

## 8. UserRoles

Asigna roles a usuarios.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| UserId | int | No | FK hacia Users.Id; parte de la PK. |
| RoleId | int | No | FK hacia Roles.Id; parte de la PK. |

### Restricciones

- Clave primaria compuesta: UserId + RoleId.
- Un usuario puede tener varios roles.
- Un rol puede estar asignado a varios usuarios.
- Retirar una asignación no elimina al usuario ni al rol.
- Las asignaciones y retiradas se registran en auditoría.
- Un administrador no puede modificar sus propias asignaciones.
- No se puede retirar el último administrador activo.

## 9. AgentProfiles

Representa la configuración básica de una identidad de agente.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ActorId | int | No | FK hacia Actors.Id; único. |
| IsEnabled | bit | No | Permite o impide nuevas acciones del agente. |

### Restricciones

- El actor relacionado debe tener ActorType = Agent.
- El agente no tiene correo ni contraseña de usuario humano.
- Sus herramientas y permisos se controlan desde la aplicación.
- Deshabilitarlo conserva sus casos, solicitudes y ejecuciones.
- Su habilitación y permisos se verifican antes de cada herramienta.

La configuración del modelo y sus credenciales se gestionan
por separado; no se guardan como datos públicos del perfil.

## 10. InventoryBalances

Conserva la existencia registrada actual y el estado de apertura
del inventario de cada producto.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| ProductId | int | No | PK y FK hacia Products.Id. No es IDENTITY. |
| QuantityOnHand | int | No | Existencia registrada, mayor o igual que cero. |
| OpeningStatus | nvarchar(20) | No | Pending o Opened. |
| RowVersion | rowversion | No | Valor generado por SQL Server para detectar cambios concurrentes. |

### Restricciones

- Al crear un producto se crea su saldo en la misma transacción,
  con cantidad cero y apertura Pending.
- ProductId como clave primaria impide dos saldos del mismo producto.
- Mientras la apertura esté pendiente, no se permiten movimientos
  operativos ni alertas de reposición.
- La aprobación de la apertura cambia OpeningStatus a Opened.
- Después de la apertura, el saldo solo cambia mediante movimientos
  autorizados y registrados en la misma transacción.
- Las cantidades pendientes de recibir no forman parte de QuantityOnHand.
- RowVersion no representa una fecha ni una cantidad de movimientos.

## 11. InventoryOpenings

Registra las solicitudes de apertura de inventario por producto
y su resolución.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ProductId | int | No | FK hacia Products.Id. |
| InitialQuantity | int | No | Cantidad física inicial; no negativa. |
| CountedAt | datetime2 | No | Momento del conteo físico, en UTC. |
| Reason | nvarchar(500) | No | Justificación de la apertura. |
| Status | nvarchar(20) | No | Pending, Applied, Rejected o Cancelled. |
| RequestedByUserId | int | No | FK hacia Users.Id. |
| RequestedAt | datetime2 | No | Fecha de solicitud, en UTC. |
| ReviewedByUserId | int | Sí | FK hacia Users.Id; gerente que resolvió. |
| ReviewedAt | datetime2 | Sí | Fecha de aprobación o rechazo. |
| ReviewComment | nvarchar(500) | Sí | Observación; obligatoria al rechazar. |
| CancelledByUserId | int | Sí | FK hacia Users.Id; responsable de cancelar. |
| CancelledAt | datetime2 | Sí | Fecha de cancelación. |
| CancellationReason | nvarchar(500) | Sí | Motivo obligatorio al cancelar. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |

### Restricciones

- Solo se solicita para productos pendientes de apertura.
- Solo puede existir una solicitud Pending por producto.
- Solo puede existir una apertura Applied por producto.
- Una solicitud rechazada o cancelada se conserva; puede presentarse
  otra mientras el producto siga pendiente de apertura.
- El encargado solicita; el gerente aprueba o rechaza.
- El solicitante o un gerente pueden cancelar una solicitud pendiente.
- Las solicitudes presentadas no se editan; se cancelan y reemplazan.
- Aprobar registra el movimiento inicial, actualiza el saldo,
  habilita el inventario y cambia la solicitud a Applied
  en una misma transacción.
- Si InitialQuantity es cero, se habilita el inventario sin generar
  un movimiento de cantidad cero.
- Los registros resueltos no pueden reabrirse.

## 12. StockCounts

Registra conteos físicos de productos cuyo inventario ya está abierto.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ProductId | int | No | FK hacia Products.Id. |
| ReferenceQuantity | int | No | Existencia registrada al abrir el conteo. |
| CountedQuantity | int | Sí | Cantidad física observada; no negativa. |
| Status | nvarchar(20) | No | Open, Confirmed o Cancelled. |
| OpenedByUserId | int | No | FK hacia Users.Id. |
| OpenedAt | datetime2 | No | Fecha de apertura en UTC. |
| ConfirmedByUserId | int | Sí | FK hacia Users.Id. |
| ConfirmedAt | datetime2 | Sí | Fecha de confirmación en UTC. |
| CancelledByUserId | int | Sí | FK hacia Users.Id. |
| CancelledAt | datetime2 | Sí | Fecha de cancelación en UTC. |
| CancellationReason | nvarchar(500) | Sí | Obligatorio al cancelar. |
| Notes | nvarchar(500) | Sí | Observaciones del conteo. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |

### Restricciones

- El producto debe tener su inventario abierto.
- Solo puede existir un conteo Open por producto.
- Abrir el conteo guarda ReferenceQuantity y bloquea las operaciones
  que modifican existencias del producto.
- La apertura se coordina con los movimientos concurrentes para
  obtener una referencia consistente.
- Mientras está abierto, CountedQuantity permanece NULL.
- El encargado que abrió el conteo confirma la cantidad observada.
- Confirmar exige una cantidad entera mayor o igual que cero.
- Confirmar guarda la cantidad observada, responsable y fecha,
  y libera el bloqueo en una misma transacción.
- El encargado que abrió el conteo o un gerente pueden cancelarlo,
  indicando el motivo y liberando el bloqueo.
- Confirmar o cancelar no modifica las existencias.
- Un conteo resuelto no puede editarse ni reabrirse.
- Si después de confirmarlo hay movimientos, no se permite aplicar
  un ajuste basado en él sin una nueva verificación.

### Diferencia

La diferencia se calcula como:

CountedQuantity - ReferenceQuantity

No se almacena como un campo independiente.

Solo tiene significado para conteos confirmados.
Una diferencia puede originar una solicitud de ajuste;
nunca modifica automáticamente el saldo.

## 13. InventoryAdjustments

Registra solicitudes de corrección de existencias y su resolución.

Cada solicitud corresponde a un producto y se origina en un conteo
confirmado o en un error de una salida registrada.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|----------------|---|---|
| Id | int            | No | PK, IDENTITY. |
| ProductId | int            | No | FK hacia Products.Id. |
| StockCountId | int            | Sí | FK hacia StockCounts.Id. |
| OriginalMovementId | bigint         | Sí | FK hacia InventoryMovements.Id. |
| QuantityDelta | int            | No | Diferencia solicitada, positiva o negativa; distinta de cero. |
| Reason | nvarchar(500)  | No | Justificación obligatoria. |
| Status | nvarchar(20)   | No | Pending, Applied, Rejected o Cancelled. |
| RequestedByUserId | int            | No | FK hacia Users.Id. |
| RequestedAt | datetime2      | No | Fecha de solicitud en UTC. |
| ReviewedByUserId | int            | Sí | FK hacia Users.Id. |
| ReviewedAt | datetime2      | Sí | Fecha de aprobación o rechazo. |
| ReviewComment | nvarchar(500)  | Sí | Observación; obligatoria al rechazar. |
| CancelledByUserId | int            | Sí | FK hacia Users.Id. |
| CancelledAt | datetime2      | Sí | Fecha de cancelación. |
| CancellationReason | nvarchar(500)  | Sí | Obligatorio al cancelar. |
| RowVersion | rowversion     | No | Control de modificaciones concurrentes. |

### Restricciones

- Debe existir exactamente un origen: StockCountId
  u OriginalMovementId, nunca ambos ni ninguno.
- El producto del ajuste debe coincidir con el producto del origen.
- Una solicitud pendiente no modifica existencias.
- El encargado solicita; el gerente aprueba o rechaza.
- El solicitante o un gerente pueden cancelar un ajuste pendiente.
- El contenido presentado no se edita; se cancela y reemplaza.
- Un ajuste basado en conteo utiliza la diferencia observada.
- Un conteo puede tener varias solicitudes históricas rechazadas
  o canceladas, pero solo una pendiente y una aplicada como máximo.
  Su diferencia no puede aplicarse dos veces.
- Si hubo movimientos posteriores al conteo, se exige una nueva
  verificación antes de continuar.
- Las correcciones de salidas consideran compensaciones anteriores
  para no devolver más unidades de las registradas originalmente.
- No se permiten ajustes que dejen existencias negativas.
- Aprobar, generar el movimiento compensatorio, actualizar el saldo
  y cambiar el estado a Applied constituye una sola transacción.
- Si la operación falla, no se confirma ninguno de esos cambios.
- Los ajustes resueltos se conservan y no pueden reabrirse.

## 14. InventoryMovements

Conserva el historial de cambios confirmados de existencias.
Cada movimiento corresponde a un producto.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | bigint | No | PK, IDENTITY. |
| ProductId | int | No | FK hacia Products.Id. |
| MovementType | nvarchar(30) | No | Tipo que determina el origen y sentido del movimiento. |
| Quantity | int | No | Cantidad entera mayor que cero. |
| OccurredAt | datetime2 | No | Momento efectivo de la operación, en UTC. |
| CreatedAt | datetime2 | No | Momento del registro, en UTC. |
| CreatedByActorId | int | No | FK hacia Actors.Id; responsable de la operación. |
| Reason | nvarchar(500) | No | Explicación del movimiento. |
| InventoryOpeningId | int | Sí | FK hacia InventoryOpenings.Id. |
| GoodsReceiptDetailId | int | Sí | FK hacia GoodsReceiptDetails.Id. |
| InventoryIssueDetailId | int | Sí | FK hacia InventoryIssueDetails.Id. |
| InventoryAdjustmentId | int | Sí | FK hacia InventoryAdjustments.Id. |
| ReversesMovementId | bigint | Sí | FK hacia InventoryMovements.Id, si corrige una salida. |

### Tipos de movimiento

| Tipo | Efecto | Origen |
|---|---|---|
| OpeningBalance | Aumenta | Apertura aprobada con cantidad mayor que cero. |
| PurchaseReceipt | Aumenta | Detalle de recepción confirmada. |
| SaleIssue | Disminuye | Detalle de salida confirmada. |
| PositiveAdjustment | Aumenta | Ajuste aprobado por diferencia de conteo. |
| NegativeAdjustment | Disminuye | Ajuste aprobado por diferencia de conteo. |
| IssueReversal | Aumenta | Ajuste aprobado que compensa una salida errónea. |

### Restricciones

- Quantity siempre es positiva; MovementType determina el efecto.
- Debe existir exactamente un documento de origen:
  apertura, detalle de recepción, detalle de salida o ajuste.
- El tipo debe corresponder al documento de origen.
- El producto debe coincidir con el del documento de origen.
- Cada documento de origen puede generar como máximo un movimiento.
- IssueReversal requiere InventoryAdjustmentId y ReversesMovementId.
  Este último es una referencia complementaria al movimiento corregido.
- En los demás tipos, ReversesMovementId debe ser NULL.
- Solo se permite compensar movimientos SaleIssue en esta versión.
- Las compensaciones acumuladas no pueden superar la cantidad original.
- Los movimientos confirmados no se editan ni eliminan.
- Registrar un movimiento y actualizar el saldo constituye una
  misma transacción.
- CreatedByActorId identifica a la persona o agente que inició la
  operación autorizada; ejecutar código automáticamente no convierte
  al agente en responsable de todas las operaciones.

## 15. InventoryIssues

Registra una salida confirmada de mercancía.
En la primera versión representa salidas por venta,
sin implementar un punto de venta completo.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| IdempotencyKey | uniqueidentifier | No | Clave única de la operación. |
| RequestHash | char(64) | No | Huella SHA-256 del contenido normalizado de la petición. |
| Reason | nvarchar(500) | No | Motivo obligatorio. |
| ExternalReference | nvarchar(100) | Sí | Referencia a un comprobante externo. |
| OccurredAt | datetime2 | No | Momento de la salida, en UTC. |
| CreatedAt | datetime2 | No | Momento del registro, en UTC. |
| CreatedByUserId | int | No | FK hacia Users.Id. |

### Restricciones

- La salida se registra y confirma en una sola operación.
- No existen borradores de salida en esta versión.
- Debe contener al menos un detalle.
- Todos los productos deben estar activos, con inventario abierto,
  sin conteos abiertos y con existencias suficientes.
- Si falla un detalle, se rechaza toda la salida.
- Cabecera, detalles, movimientos y saldos se guardan
  en una misma transacción.
- Una salida confirmada no se edita ni elimina.
- Las correcciones requieren un ajuste aprobado.
- La misma clave con el mismo contenido devuelve el resultado
  existente, sin duplicar movimientos.
- La misma clave con contenido diferente se rechaza por conflicto.
- La autorización se comprueba también en los reintentos.
- RequestHash lo calcula el servidor; no se confía en una huella
  enviada por el cliente.

## 16. InventoryIssueDetails

Identifica los productos y cantidades de una salida.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| InventoryIssueId | int | No | FK hacia InventoryIssues.Id. |
| ProductId | int | No | FK hacia Products.Id. |
| Quantity | int | No | Cantidad entera mayor que cero. |

### Restricciones

- La combinación InventoryIssueId + ProductId es única.
- Cada detalle genera exactamente un movimiento SaleIssue.
- El movimiento conserva el mismo producto y cantidad del detalle.
- Las compensaciones posteriores no modifican el detalle original.

## 17. ReplenishmentRequests

Representa una propuesta de compra presentada para revisión.
Cada solicitud corresponde a un único proveedor.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| SupplierId | int | No | FK hacia Suppliers.Id. |
| Status | nvarchar(20) | No | Pending, Approved, Rejected, Cancelled o ConvertedToOrder. |
| Justification | nvarchar(1000) | No | Justificación general de la propuesta. |
| CreatedByActorId | int | No | FK hacia Actors.Id; persona o agente creador. |
| CreatedAt | datetime2 | No | Fecha de presentación en UTC. |
| ReviewedByUserId | int | Sí | FK hacia Users.Id; gerente que revisó. |
| ReviewedAt | datetime2 | Sí | Fecha de aprobación o rechazo. |
| ReviewComment | nvarchar(500) | Sí | Observación; obligatoria al rechazar. |
| CancelledByUserId | int | Sí | FK hacia Users.Id. |
| CancelledAt | datetime2 | Sí | Fecha de cancelación. |
| CancellationReason | nvarchar(500) | Sí | Obligatorio al cancelar. |
| ReplacesRequestId | int | Sí | FK hacia ReplenishmentRequests.Id; propuesta anterior. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |
| IdempotencyKey | uniqueidentifier | No | Clave única de la operación de creación. |
| RequestHash | char(64) | No | Huella del contenido normalizado, calculada por el servidor. |

### Restricciones

- Se crea en estado Pending y contiene al menos un detalle.
- Su contenido no se edita después de presentarse.
- Solo el gerente puede aprobar o rechazar una solicitud pendiente.
- Un encargado o gerente puede cancelarla antes de generar una orden.
- El agente no puede aprobar, rechazar ni cancelar solicitudes.
- Una solicitud rechazada o cancelada puede tener un reemplazo.
- ReplacesRequestId no puede referenciar la propia solicitud.
- Presentar un reemplazo no transfiere la aprobación anterior.
- Una solicitud aprobada puede generar como máximo una orden.
- Emitir la orden y cambiar el estado a ConvertedToOrder
  constituye una misma transacción.
- Ningún estado de la solicitud modifica por sí solo las existencias
  ni las cantidades pendientes de recibir.
- Los estados Rejected, Cancelled y ConvertedToOrder son finales.

## 18. ReplenishmentRequestDetails

Conserva los productos propuestos y las condiciones utilizadas
para evaluar la compra.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ReplenishmentRequestId | int | No | FK hacia ReplenishmentRequests.Id. |
| ProductSupplierId | int | No | FK hacia ProductSuppliers.Id. |
| EstimatedNeedQuantity | int | No | Necesidad estimada, mayor que cero. |
| ProposedQuantity | int | No | Cantidad propuesta para comprar, mayor que cero. |
| EstimatedUnitCost | decimal(18,2) | No | Copia del costo evaluado, mayor que cero. |
| LeadTimeDaysSnapshot | int | No | Copia del plazo estimado evaluado. |
| MinimumOrderQuantitySnapshot | int | No | Copia de la compra mínima evaluada. |
| Justification | nvarchar(1000) | No | Explicación de la cantidad y elección propuesta. |

### Restricciones

- ReplenishmentRequestId + ProductSupplierId es una combinación única.
- El proveedor de ProductSupplierId debe coincidir con el proveedor
  de la cabecera.
- El producto se obtiene mediante ProductSuppliers.ProductId.
- Al presentar la solicitud, producto, proveedor y relación
  de suministro deben estar activos.
- ProposedQuantity debe ser mayor o igual que EstimatedNeedQuantity
  y MinimumOrderQuantitySnapshot.
- Los valores históricos no cambian al actualizar ProductSuppliers.
- Antes de emitir una orden se comparan las condiciones actuales
  con las aprobadas. Las diferencias exigen una nueva revisión.
- El subtotal se calcula como ProposedQuantity por EstimatedUnitCost.
- El excedente se calcula como ProposedQuantity menos
  EstimatedNeedQuantity.
- El total estimado de la solicitud es la suma de sus subtotales.
- En esta versión, el total estimado representa únicamente el costo
  de los productos; no calcula flete, impuestos ni costos de importación.

## 19. PurchaseOrders

Representa una orden de compra emitida a partir de una solicitud aprobada.
La emisión es simulada en la primera versión.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ReplenishmentRequestId | int | No | FK hacia ReplenishmentRequests.Id; único. |
| SupplierId | int | No | FK hacia Suppliers.Id. |
| Status | nvarchar(30) | No | Estado de la orden. |
| IssuedByUserId | int | No | FK hacia Users.Id; gerente que emitió. |
| IssuedAt | datetime2 | No | Fecha de emisión en UTC. |
| Notes | nvarchar(1000) | Sí | Observaciones de la orden. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |

### Estados

- Issued: emitida, sin recepciones confirmadas y con saldo pendiente.
- PartiallyReceived: parcialmente recibida, con saldo pendiente.
- Completed: toda la cantidad ordenada fue recibida, sin cancelaciones.
- Cancelled: toda la cantidad fue cancelada, sin recepciones.
- ClosedPartial: sin saldo pendiente, con cantidades recibidas y canceladas.

### Restricciones

- La solicitud de origen debe estar Approved.
- SupplierId debe coincidir con el proveedor de la solicitud.
- Solo el gerente puede emitir la orden.
- Se verifican las condiciones actuales frente a las aprobadas.
- Cada solicitud genera como máximo una orden.
- La emisión copia las condiciones aprobadas a los detalles de la orden.
- Crear cabecera y detalles, y cambiar la solicitud a ConvertedToOrder,
  constituye una misma transacción.
- Emitir aumenta las cantidades pendientes de recibir;
  no modifica las existencias.
- Las condiciones emitidas no se sobrescriben con cambios del catálogo.
- Las cancelaciones posteriores afectan únicamente cantidades pendientes
  y conservan motivo, fecha y responsable.

## 20. PurchaseOrderDetails

Conserva los productos y condiciones acordadas al emitir una orden,
junto con las cantidades recibidas y canceladas.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| PurchaseOrderId | int | No | FK hacia PurchaseOrders.Id. |
| ReplenishmentRequestDetailId | int | No | FK hacia ReplenishmentRequestDetails.Id; único. |
| ProductId | int | No | FK hacia Products.Id. |
| OrderedQuantity | int | No | Cantidad ordenada, mayor que cero. |
| ReceivedQuantity | int | No | Cantidad recibida acumulada; comienza en cero. |
| CancelledQuantity | int | No | Cantidad cancelada acumulada; comienza en cero. |
| UnitCost | decimal(18,2) | No | Costo unitario acordado, mayor que cero. |
| LeadTimeDays | int | No | Plazo estimado acordado; no negativo. |
| MinimumOrderQuantity | int | No | Compra mínima evaluada al emitir. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |

### Restricciones

- PurchaseOrderId + ProductId es una combinación única.
- El detalle de solicitud debe pertenecer a la solicitud
  que originó la orden.
- ProductId debe coincidir con el producto del detalle de solicitud.
- OrderedQuantity y las condiciones se copian de la propuesta aprobada.
- Las cantidades recibidas y canceladas no pueden ser negativas.
- ReceivedQuantity + CancelledQuantity no puede superar OrderedQuantity.
- Solo las recepciones confirmadas aumentan ReceivedQuantity.
- Solo una cancelación autorizada aumenta CancelledQuantity.
- Los acumulados se actualizan junto con los documentos que
  los justifican, en la misma transacción.
- Las cantidades ordenadas y condiciones emitidas no se editan.
- Una cancelación no cambia OrderedQuantity: conserva lo originalmente
  pedido y registra por separado lo que se decidió cancelar.
- Las cantidades pendientes no se almacenan en otro campo;
  se calculan a partir de las cantidades anteriores.

### Cálculos

PendingQuantity =
OrderedQuantity - ReceivedQuantity - CancelledQuantity

OrderedSubtotal =
OrderedQuantity * UnitCost


## 21. PurchaseOrderCancellations

Registra cancelaciones autorizadas de cantidades pendientes
de un detalle de orden.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| PurchaseOrderDetailId | int | No | FK hacia PurchaseOrderDetails.Id. |
| Quantity | int | No | Cantidad cancelada, mayor que cero. |
| Reason | nvarchar(500) | No | Motivo obligatorio. |
| CancelledByUserId | int | No | FK hacia Users.Id; gerente responsable. |
| CancelledAt | datetime2 | No | Fecha en UTC. |
| IdempotencyKey | uniqueidentifier | No | Clave única de la operación. |
| RequestHash | char(64) | No | Huella del contenido normalizado, calculada por el servidor. |

### Restricciones

- Solo el gerente puede cancelar cantidades pendientes.
- Quantity no puede superar el pendiente actual del detalle.
- La cancelación no reduce ReceivedQuantity ni las existencias.
- Registrar la cancelación, actualizar CancelledQuantity y recalcular
  el estado de la orden constituye una misma transacción.
- Los registros confirmados no se editan ni eliminan.
- Reintentar con la misma clave y contenido devuelve el resultado
  existente; con contenido diferente se rechaza.
- La autorización se verifica también en los reintentos.

## 22. GoodsReceipts

Registra una entrega física asociada a una orden de compra.
Su confirmación incorpora la mercancía al inventario.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| PurchaseOrderId | int | No | FK hacia PurchaseOrders.Id. |
| Status | nvarchar(20) | No | Pending, Confirmed o Cancelled. |
| SupplierDocumentReference | nvarchar(100) | Sí | Referencia del documento de entrega. |
| ArrivedAt | datetime2 | No | Momento de llegada, en UTC. |
| RegisteredByUserId | int | No | FK hacia Users.Id. |
| CreatedAt | datetime2 | No | Momento de registro, en UTC. |
| ConfirmedByUserId | int | Sí | FK hacia Users.Id. |
| ConfirmedAt | datetime2 | Sí | Momento de confirmación, en UTC. |
| CancelledByUserId | int | Sí | FK hacia Users.Id. |
| CancelledAt | datetime2 | Sí | Momento de cancelación, en UTC. |
| CancellationReason | nvarchar(500) | Sí | Obligatorio al cancelar. |
| Notes | nvarchar(1000) | Sí | Observaciones de la entrega. |
| IdempotencyKey | uniqueidentifier | No | Clave única del registro de llegada. |
| RequestHash | char(64) | No | Huella del contenido normalizado, calculada por el servidor. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |

### Restricciones

- Se crea en estado Pending y contiene al menos un detalle.
- Solo se registra contra una orden con cantidades pendientes.
- Registrar la llegada no genera movimientos, no aumenta existencias
  ni reduce el pendiente de la orden.
- La llegada puede registrarse aunque haya conteos abiertos.
  La mercancía permanece físicamente separada.
- Los detalles presentados no se editan. Si contienen errores,
  se cancela la recepción pendiente y se registra otra.
- Un encargado o gerente puede cancelar una recepción pendiente,
  indicando el motivo.
- El encargado confirma la recepción tras verificar la entrega.
- Antes de confirmar se validan nuevamente los pendientes de la orden,
  la apertura de inventario y la ausencia de conteos abiertos.
- Si cualquier detalle incumple las condiciones, se rechaza
  toda la confirmación y la recepción permanece Pending.
- Confirmar actualiza detalles de orden, movimientos, saldos
  y estado de recepción en una misma transacción.
- Una recepción Confirmed o Cancelled no puede reabrirse.
- Una recepción confirmada no admite correcciones en esta versión.
- La clave de idempotencia evita duplicar el registro de llegada:
  misma clave y contenido devuelve el registro existente;
  misma clave con contenido diferente se rechaza.
- Repetir la confirmación de una recepción ya confirmada devuelve
  su resultado sin generar movimientos nuevos.
- La autorización se verifica también en los reintentos.

## 23. GoodsReceiptDetails

Registra las cantidades declaradas en una entrega,
vinculadas a los detalles de la orden.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| GoodsReceiptId | int | No | FK hacia GoodsReceipts.Id. |
| PurchaseOrderDetailId | int | No | FK hacia PurchaseOrderDetails.Id. |
| Quantity | int | No | Cantidad entera mayor que cero. |

### Restricciones

- GoodsReceiptId + PurchaseOrderDetailId es una combinación única.
- El detalle de orden debe pertenecer a la orden de la cabecera.
- El producto se obtiene mediante PurchaseOrderDetails.ProductId.
- La cantidad declarada no puede superar el pendiente al registrar.
- Una recepción pendiente no reserva cantidades de la orden.
  El límite se valida nuevamente al confirmar.
- El sistema no reduce automáticamente la cantidad declarada.
- Cada detalle de una recepción confirmada genera exactamente
  un movimiento PurchaseReceipt con el mismo producto y cantidad.
- Los detalles pendientes o cancelados no generan movimientos.
## 24. StockAlerts

Registra episodios de existencias bajas por producto.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ProductId | int | No | FK hacia Products.Id. |
| Status | nvarchar(20) | No | Active, Resolved o Cancelled. |
| DetectedQuantity | int | No | Saldo observado al detectar la alerta. |
| MinimumStockSnapshot | int | No | Nivel mínimo utilizado al detectarla. |
| CreatedAt | datetime2 | No | Fecha de detección, en UTC. |
| ClosedAt | datetime2 | Sí | Fecha de resolución o cancelación. |
| ClosureReason | nvarchar(500) | Sí | Explicación del cierre. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |

### Restricciones

- Solo se generan para productos activos con inventario abierto.
- La detección ocurre cuando QuantityOnHand es menor o igual
  que MinimumStock.
- Solo puede existir una alerta Active por producto.
- Las revisiones repetidas no crean alertas activas duplicadas.
- Los valores de detección se conservan como evidencia histórica.
- La alerta se resuelve cuando el saldo supera el mínimo vigente.
- Aprobar una solicitud o emitir una orden no resuelve la alerta:
  las existencias todavía no aumentaron.
- Si se desactiva el producto, su alerta activa se cancela
  con el motivo correspondiente.
- Una nueva caída después del cierre genera otra alerta.
- La detección y seguimiento manual siguen disponibles
  aunque el agente esté deshabilitado.

## 25. AgentCases

Conserva el seguimiento operativo de una alerta entre ejecuciones
del agente.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| StockAlertId | int | No | FK hacia StockAlerts.Id; único. |
| AgentProfileId | int | No | FK hacia AgentProfiles.Id. |
| Status | nvarchar(30) | No | Estado del caso. |
| CurrentRequestId | int | Sí | FK hacia ReplenishmentRequests.Id. |
| Summary | nvarchar(2000) | Sí | Resumen operativo actualizado. |
| CreatedAt | datetime2 | No | Fecha de creación en UTC. |
| UpdatedAt | datetime2 | No | Última actualización en UTC. |
| ClosedAt | datetime2 | Sí | Fecha de cierre en UTC. |
| ClosureReason | nvarchar(500) | Sí | Motivo del cierre. |
| RowVersion | rowversion | No | Control de modificaciones concurrentes. |

### Estados

- Open: pendiente de investigación o de continuar el trabajo.
- AwaitingApproval: espera una decisión sobre una solicitud.
- Monitoring: sigue una reposición abierta.
- NeedsHumanReview: requiere intervención para poder continuar.
- Closed: seguimiento finalizado con motivo registrado.

### Restricciones

- Una alerta puede tener como máximo un caso.
- El producto se obtiene mediante la alerta relacionada.
- El caso conserva su estado aunque no haya una ejecución activa.
- CurrentRequestId identifica la propuesta que se sigue actualmente.
  Las propuestas anteriores conservan su historial y vínculos de reemplazo.
- La solicitud relacionada debe incluir el producto de la alerta.
- El resumen no sustituye las consultas a los datos actuales.
- Una decisión del gerente no autoriza al agente a omitir validaciones.
- Si la solicitud es rechazada, el caso requiere revisión humana
  antes de presentar automáticamente otra propuesta.
- Resolver la alerta no cierra automáticamente el caso si todavía
  existe una compra relacionada pendiente de seguimiento.
- El caso puede cerrarse cuando la alerta está resuelta y no quedan
  decisiones ni compras relacionadas pendientes.
- Un gerente puede cerrar el caso con una justificación, incluso
  si el problema continúa. Esto no resuelve la alerta.
- Deshabilitar al agente conserva el caso y bloquea nuevas acciones
  del agente; no elimina tareas pendientes.
- Un caso Closed no se reabre automáticamente.

## 26. AgentRuns

Registra cada intento del agente de trabajar en un caso.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | bigint | No | PK, IDENTITY. |
| AgentCaseId | int | No | FK hacia AgentCases.Id. |
| AgentProfileId | int | No | FK hacia AgentProfiles.Id. |
| Status | nvarchar(20) | No | Running, Completed, Failed, Stopped o Interrupted. |
| StartedAt | datetime2 | No | Inicio en UTC. |
| FinishedAt | datetime2 | Sí | Fin en UTC. |
| ModelIdentifier | nvarchar(200) | No | Modelo utilizado. |
| WorkflowVersion | nvarchar(50) | No | Versión del flujo e instrucciones utilizadas. |
| MaxSteps | int | No | Límite de pasos para esta ejecución. |
| TimeoutSeconds | int | No | Tiempo máximo configurado. |
| ResultSummary | nvarchar(2000) | Sí | Resultado operativo verificable. |
| ErrorCode | nvarchar(100) | Sí | Código de error, si corresponde. |

### Restricciones

- El agente debe estar habilitado al iniciar.
- Solo puede existir una ejecución Running por caso.
- La ejecución utiliza el agente asignado al caso.
- Los límites deben ser mayores que cero.
- Completed significa que el intento terminó correctamente;
  no implica que el caso esté cerrado.
- Esperar una aprobación finaliza la ejecución y deja el estado
  correspondiente en el caso.
- Failed indica un error que impidió completar el intento.
- Stopped indica una detención controlada, por límite o deshabilitación.
- Interrupted identifica una ejecución abandonada por una interrupción
  del proceso, detectada durante la recuperación.
- Antes de reintentar se revisan los efectos ya confirmados.
- Un fallo no revierte automáticamente acciones confirmadas
  en transacciones anteriores.

## 27. AgentToolCalls

Registra las herramientas solicitadas durante una ejecución
y el resultado de su procesamiento.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | bigint | No | PK, IDENTITY. |
| AgentRunId | bigint | No | FK hacia AgentRuns.Id. |
| SequenceNumber | int | No | Orden de la llamada dentro de la ejecución. |
| ToolName | nvarchar(100) | No | Nombre de la herramienta solicitada. |
| Status | nvarchar(20) | No | Started, Succeeded, Failed, Denied o Unknown. |
| ArgumentsJson | nvarchar(max) | No | Argumentos registrados sin secretos. |
| ResultSummary | nvarchar(2000) | Sí | Resumen del resultado. |
| RequestedAt | datetime2 | No | Momento de la solicitud en UTC. |
| FinishedAt | datetime2 | Sí | Momento de finalización en UTC. |
| OperationKey | uniqueidentifier | Sí | Clave de idempotencia para acciones de escritura. |
| ErrorCode | nvarchar(100) | Sí | Motivo técnico o de autorización. |

### Restricciones

- AgentRunId + SequenceNumber es una combinación única.
- SequenceNumber debe ser mayor que cero.
- ArgumentsJson debe contener JSON válido.
- Solo se ejecutan herramientas incluidas en la lista autorizada.
- Antes de ejecutar se validan argumentos, permisos y habilitación
  actual del agente.
- Denied registra una llamada rechazada sin ejecutar la herramienta.
- Succeeded registra una herramienta completada correctamente.
- Failed registra un error con resultado conocido.
- Unknown indica que no se pudo determinar si la acción se confirmó.
  Debe reconciliarse antes de repetirla.
- Las herramientas de escritura deben admitir idempotencia.
  Sus reintentos reutilizan OperationKey para la misma operación.
- Las claves no se generan de nuevo únicamente por reintentar.
- Los registros no incluyen contraseñas, tokens ni razonamientos
  internos del modelo.

## 28. AuditLogs

Conserva eventos relevantes de modificación y decisiones
realizadas en el sistema.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | bigint | No | PK, IDENTITY. |
| ActorId | int | Sí | FK hacia Actors.Id; responsable, cuando existe. |
| EventSource | nvarchar(20) | No | User, Agent o System. |
| Action | nvarchar(100) | No | Acción registrada. |
| EntityType | nvarchar(100) | No | Tipo de registro afectado. |
| EntityId | nvarchar(100) | No | Identificador del registro afectado. |
| PreviousValuesJson | nvarchar(max) | Sí | Valores anteriores relevantes, sin secretos. |
| NewValuesJson | nvarchar(max) | Sí | Valores nuevos relevantes, sin secretos. |
| Reason | nvarchar(1000) | Sí | Justificación, cuando corresponda. |
| OccurredAt | datetime2 | No | Fecha del evento en UTC. |
| CorrelationId | uniqueidentifier | No | Identificador para relacionar eventos de una operación. |

### Restricciones

- Los eventos confirmados no se editan ni eliminan mediante
  las funciones normales de la aplicación.
- Los cambios de negocio y su auditoría se guardan en la misma
  transacción cuando pertenecen a la misma base de datos.
- Los valores JSON deben ser válidos cuando estén presentes.
- Solo se registran los campos necesarios para explicar el cambio.
- No se guardan contraseñas, hashes de contraseñas, tokens
  ni claves privadas.
- User y Agent requieren ActorId del tipo correspondiente.
- System permite ActorId NULL para procesos deterministas
  que no representan a una persona ni a un agente.
- EntityId es una referencia descriptiva, no una clave foránea.
- La auditoría complementa los documentos de negocio;
  no sustituye movimientos, solicitudes ni aprobaciones.
- CorrelationId relaciona eventos; no sustituye una clave
  de idempotencia.

## 29. KnowledgeDocuments

Conserva versiones de documentos utilizados como conocimiento:
políticas internas y condiciones documentales de proveedores.

Cada fila representa una versión concreta de un documento.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| DocumentKey | uniqueidentifier | No | Identifica el documento entre versiones. |
| VersionNumber | int | No | Versión entera mayor que cero. |
| Title | nvarchar(200) | No | Título del documento. |
| SourceReference | nvarchar(500) | No | Referencia al archivo o fuente conservada. |
| ContentHash | char(64) | No | Huella del contenido. |
| Status | nvarchar(20) | No | Processing, Ready, Failed o Retired. |
| IsCurrent | bit | No | Indica la versión vigente. |
| UploadedByUserId | int | No | FK hacia Users.Id. |
| CreatedAt | datetime2 | No | Fecha de incorporación en UTC. |
| EmbeddingModel | nvarchar(200) | Sí | Modelo utilizado para generar embeddings. |
| EmbeddingDimensions | int | Sí | Número de componentes del vector. |

### Restricciones

- DocumentKey + VersionNumber es una combinación única.
- Solo puede existir una versión vigente por DocumentKey.
- Una versión nueva se prepara antes de sustituir a la vigente.
- Solo se recuperan fragmentos de versiones Ready y vigentes.
- El contenido de una versión indexada no se sobrescribe.
- Las versiones anteriores se conservan para auditoría.
- Solo el administrador incorpora o retira documentos.
- El contenido documental no puede otorgar permisos al agente.

## 30. KnowledgeChunks

Conserva los fragmentos de texto y su referencia al índice vectorial.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | bigint | No | PK, IDENTITY. |
| KnowledgeDocumentId | int | No | FK hacia KnowledgeDocuments.Id. |
| ChunkIndex | int | No | Posición del fragmento, desde cero. |
| Content | nvarchar(max) | No | Texto del fragmento. |
| SourceLocator | nvarchar(200) | Sí | Página o sección de origen. |
| VectorPointId | uniqueidentifier | No | Identificador único en el índice vectorial. |

### Restricciones

- KnowledgeDocumentId + ChunkIndex es una combinación única.
- El texto del fragmento queda asociado a una versión inmutable.
- Los embeddings se almacenan en el índice vectorial.
- SQL Server conserva texto, metadatos y referencias.
- Una versión solo pasa a Ready cuando todos sus fragmentos
  están indexados correctamente.
- Las consultas usan embeddings compatibles con el modelo
  y dimensiones del índice.
- Los resultados vectoriales se verifican contra el estado
  vigente del documento antes de entregarlos al agente.
- Los fragmentos utilizados como evidencia histórica se conservan.

## 31. AgentEvidence

Relaciona las ejecuciones con los fragmentos recuperados
y distingue cuáles se citaron en su resultado.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | bigint | No | PK, IDENTITY. |
| AgentRunId | bigint | No | FK hacia AgentRuns.Id. |
| KnowledgeChunkId | bigint | No | FK hacia KnowledgeChunks.Id. |
| RetrievedAt | datetime2 | No | Fecha de recuperación en UTC. |
| WasCited | bit | No | Indica si se citó como respaldo del resultado. |

### Restricciones

- AgentRunId + KnowledgeChunkId es una combinación única.
- Recuperar un fragmento no implica que se haya utilizado como respaldo.
- La evidencia identifica la versión exacta consultada.
- Retirar un documento de futuras búsquedas no elimina
  sus referencias históricas.
- Las existencias, precios y permisos actuales se consultan
  mediante herramientas; no se obtienen de documentos como
  sustituto de los datos operativos.
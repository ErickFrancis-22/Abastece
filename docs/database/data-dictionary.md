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
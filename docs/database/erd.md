# Diagrama entidad-relación — Abastece

Estado: diseño inicial de catálogo, identidades y acceso.
Los campos completos se describen en data-dictionary.md.

## 1. Catálogo y proveedores

```mermaid
erDiagram
    Categories ||--o{ Products : agrupa
    Products ||--o{ ProductSuppliers : tiene
    Suppliers ||--o{ ProductSuppliers : ofrece

    Categories {
        int Id PK
        nvarchar Name UK
        bit IsActive
    }

    Products {
        int Id PK
        int CategoryId FK
        nvarchar Sku UK
        nvarchar Name
        int MinimumStock
        int TargetStock
        char CountryOfOriginCode
        nvarchar ImageUrl
        bit IsActive
    }

    Suppliers {
        int Id PK
        nvarchar Name
        char CountryCode
        bit IsActive
    }

    ProductSuppliers {
        int Id PK
        int ProductId FK
        int SupplierId FK
        decimal UnitCost
        int LeadTimeDays
        int MinimumOrderQuantity
        datetime2 UpdatedAt
        bit IsActive
    }
```

### Restricciones adicionales

- ProductId + SupplierId es una combinación única.
- TargetStock debe ser mayor que MinimumStock.
- Los costos deben ser mayores que cero.
- Cada producto pertenece a una categoría.
- Un producto puede existir sin proveedores asociados.

## 2. Identidades y perfiles

```mermaid
erDiagram
    Actors ||--o| Users : identifica
    Actors ||--o| AgentProfiles : identifica

    Actors {
        int Id PK
        nvarchar ActorType
        nvarchar DisplayName
        datetime2 CreatedAt
    }

    Users {
        int Id PK
        int ActorId FK, UK
        nvarchar Email
        nvarchar NormalizedEmail UK
        nvarchar PasswordHash
        bit IsActive
        datetime2 CreatedAt
    }

    AgentProfiles {
        int Id PK
        int ActorId FK, UK
        bit IsEnabled
    }
```

### Restricciones adicionales

- Cada actor corresponde exactamente a un usuario o a un agente,
  nunca a ambos.
- ActorType debe coincidir con el perfil relacionado.
- El actor y su perfil se crean en una misma transacción.
- El diagrama muestra las relaciones individuales; la exclusividad
  y la existencia de un perfil requieren validaciones adicionales.

## 3. Asignación de roles

Users es la misma entidad presentada en el diagrama anterior.

```mermaid
erDiagram
    Users ||--o{ UserRoles : recibe
    Roles ||--o{ UserRoles : asigna

    Users {
        int Id PK
        int ActorId FK, UK
    }

    Roles {
        int Id PK
        nvarchar Name UK
    }

    UserRoles {
        int UserId PK, FK
        int RoleId PK, FK
    }
```

### Restricciones adicionales

- UserRoles tiene una clave primaria compuesta por UserId + RoleId.
- Un usuario puede tener cero o varios roles.
- Tener una cuenta sin roles no concede permisos de negocio.
- Los roles son predefinidos en la primera versión.

## 4. Saldo y apertura de inventario

```mermaid
erDiagram
    Products ||--|| InventoryBalances : tiene
    Products ||--o{ InventoryOpenings : registra
    Users ||--o{ InventoryOpenings : solicita

    Products {
        int Id PK
    }

    InventoryBalances {
        int ProductId PK, FK
        int QuantityOnHand
        nvarchar OpeningStatus
        rowversion RowVersion
    }

    InventoryOpenings {
        int Id PK
        int ProductId FK
        int InitialQuantity
        datetime2 CountedAt
        nvarchar Status
        int RequestedByUserId FK
        int ReviewedByUserId FK
        int CancelledByUserId FK
        rowversion RowVersion
    }

    Users {
        int Id PK
    }
```

### Restricciones adicionales

- Cada producto tiene exactamente una fila de saldo, creada
  en la misma transacción que el producto.
- Un producto puede tener varias solicitudes de apertura históricas.
- Solo puede tener una solicitud pendiente y una apertura aplicada
  como máximo.
- RequestedByUserId identifica al solicitante.
- ReviewedByUserId es opcional mientras no exista una revisión.
- CancelledByUserId es opcional y solo se informa al cancelar.
- Los tres campos de responsable referencian Users.Id.
- Para simplificar el dibujo, se muestra únicamente la relación
  del solicitante con Users.
- Los campos completos se describen en data-dictionary.md.

## 5. Conteos físicos

```mermaid
erDiagram
    Products ||--o{ StockCounts : tiene
    Users ||--o{ StockCounts : abre

    Products {
        int Id PK
    }

    Users {
        int Id PK
    }

    StockCounts {
        int Id PK
        int ProductId FK
        int ReferenceQuantity
        int CountedQuantity
        nvarchar Status
        int OpenedByUserId FK
        datetime2 OpenedAt
        int ConfirmedByUserId FK
        datetime2 ConfirmedAt
        int CancelledByUserId FK
        datetime2 CancelledAt
        rowversion RowVersion
    }
```

Los responsables de apertura, confirmación y cancelación
referencian Users.Id. Solo se dibuja la relación de apertura
para simplificar la vista.

Solo puede existir un conteo abierto por producto.
Los campos completos y restricciones están en el diccionario.

## 6. Ajustes de inventario

```mermaid
erDiagram
    Products ||--o{ InventoryAdjustments : tiene
    StockCounts o|--o{ InventoryAdjustments : fundamenta
    InventoryMovements o|--o{ InventoryAdjustments : origina
    Users ||--o{ InventoryAdjustments : solicita

    Products {
        int Id PK
    }

    StockCounts {
        int Id PK
        int ProductId FK
    }

    InventoryMovements {
        int Id PK
        int ProductId FK
    }

    Users {
        int Id PK
    }

    InventoryAdjustments {
        int Id PK
        int ProductId FK
        int StockCountId FK
        int OriginalMovementId FK
        int QuantityDelta
        nvarchar Status
        int RequestedByUserId FK
        int ReviewedByUserId FK
        int CancelledByUserId FK
        rowversion RowVersion
    }
```

### Restricciones adicionales

- Cada ajuste referencia exactamente un origen: conteo o movimiento.
- El origen debe corresponder al mismo producto del ajuste.
- Un conteo puede tener solicitudes históricas rechazadas o canceladas,
  pero como máximo una pendiente y una aplicada.
- Un movimiento puede tener varias correcciones, siempre respetando
  la cantidad máxima que puede compensarse.
- Las relaciones opcionales del dibujo no expresan por sí solas
  la exclusividad entre ambos orígenes.
- Solo se dibuja la relación del solicitante para simplificar.
  Revisor y responsable de cancelación también referencian Users.Id.
- InventoryMovements se presenta como referencia parcial;
  su estructura completa se documentará posteriormente.

## 7. Movimientos de inventario

```mermaid
erDiagram
    Products ||--o{ InventoryMovements : registra
    Actors ||--o{ InventoryMovements : realiza
    InventoryOpenings o|--o| InventoryMovements : origina
    InventoryAdjustments o|--o| InventoryMovements : origina

    Products {
        int Id PK
    }

    Actors {
        int Id PK
    }

    InventoryOpenings {
        int Id PK
    }

    InventoryAdjustments {
        int Id PK
        bigint OriginalMovementId FK
    }

    InventoryMovements {
        bigint Id PK
        int ProductId FK
        nvarchar MovementType
        int Quantity
        datetime2 OccurredAt
        datetime2 CreatedAt
        int CreatedByActorId FK
        int InventoryOpeningId FK
        int GoodsReceiptDetailId FK
        int InventoryIssueDetailId FK
        int InventoryAdjustmentId FK
        bigint ReversesMovementId FK
    }
```

### Restricciones adicionales

- Cada movimiento tiene un producto, un actor y exactamente
  un documento de origen.
- Los documentos de origen posibles son apertura, detalle
  de recepción, detalle de salida o ajuste.
- Cada documento de origen genera como máximo un movimiento.
- Una apertura con cero unidades no genera movimiento.
- Un ajuste aplicado genera exactamente un movimiento.
- Las relaciones con detalles de recepción y salida se dibujarán
  al incorporar esas entidades.
- OriginalMovementId identifica el movimiento que motivó
  una solicitud de corrección.
- ReversesMovementId identifica el movimiento compensado
  por una corrección aplicada.
- Las referencias al movimiento original y las restricciones
  entre tipos y orígenes se describen en el diccionario.
- Para simplificar esta vista, no se dibujan las relaciones
  al movimiento original.


## 8. Salidas de inventario

```mermaid
erDiagram
    Users ||--o{ InventoryIssues : registra
    InventoryIssues ||--|{ InventoryIssueDetails : contiene
    Products ||--o{ InventoryIssueDetails : sale_en
    InventoryIssueDetails o|--|| InventoryMovements : genera

    Users {
        int Id PK
    }

    Products {
        int Id PK
    }

    InventoryIssues {
        int Id PK
        uniqueidentifier IdempotencyKey UK
        char RequestHash
        nvarchar Reason
        nvarchar ExternalReference
        datetime2 OccurredAt
        datetime2 CreatedAt
        int CreatedByUserId FK
    }

    InventoryIssueDetails {
        int Id PK
        int InventoryIssueId FK
        int ProductId FK
        int Quantity
    }

    InventoryMovements {
        bigint Id PK
        int InventoryIssueDetailId FK
        nvarchar MovementType
        int Quantity
    }
```

### Restricciones adicionales

- Una salida confirmada contiene uno o varios detalles.
- InventoryIssueId + ProductId es una combinación única.
- Cada detalle genera exactamente un movimiento SaleIssue.
- Un movimiento puede no tener detalle de salida porque puede
  originarse en una apertura, recepción o ajuste.
- InventoryMovements.InventoryIssueDetailId es opcional y único
  cuando tiene valor.
- Cabecera, detalles, movimientos y actualización de saldos
  se confirman en una misma transacción.
- No se permiten salidas parciales implícitas.


## 9. Solicitudes de reposición

```mermaid
erDiagram
    Suppliers ||--o{ ReplenishmentRequests : recibe_propuesta
    Actors ||--o{ ReplenishmentRequests : crea
    ReplenishmentRequests ||--|{ ReplenishmentRequestDetails : contiene
    ProductSuppliers ||--o{ ReplenishmentRequestDetails : fundamenta

    Suppliers {
        int Id PK
    }

    Actors {
        int Id PK
    }

    ProductSuppliers {
        int Id PK
        int ProductId FK
        int SupplierId FK
    }

    ReplenishmentRequests {
        int Id PK
        int SupplierId FK
        nvarchar Status
        int CreatedByActorId FK
        int ReviewedByUserId FK
        int CancelledByUserId FK
        int ReplacesRequestId FK
        rowversion RowVersion
    }

    ReplenishmentRequestDetails {
        int Id PK
        int ReplenishmentRequestId FK
        int ProductSupplierId FK
        int EstimatedNeedQuantity
        int ProposedQuantity
        decimal EstimatedUnitCost
        int LeadTimeDaysSnapshot
        int MinimumOrderQuantitySnapshot
    }
```

### Restricciones adicionales

- Cada solicitud pertenece a un proveedor y contiene al menos un detalle.
- El proveedor de cada ProductSupplierId debe coincidir con el
  SupplierId de la cabecera.
- ReplenishmentRequestId + ProductSupplierId es una combinación única.
- CreatedByActorId permite identificar a un creador humano o agente.
- ReviewedByUserId y CancelledByUserId referencian Users.Id.
- ReplacesRequestId referencia una solicitud anterior de la misma tabla.
- Las relaciones de revisión, cancelación y reemplazo no se dibujan
  para simplificar esta vista.
- Las condiciones copiadas en el detalle conservan lo presentado
  al gerente, aunque cambie el catálogo.
- Los campos completos se describen en el diccionario de datos.
```



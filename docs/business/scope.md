# Alcance del proyecto — Abastece

## 1. Descripción

Abastece es un sistema empresarial de inventario y operaciones para
una distribuidora de repuestos y suministros.

Centraliza productos, proveedores, movimientos de inventario y compras.
Incluye un agente de inteligencia artificial que investiga riesgos de
desabastecimiento y gestiona solicitudes de reposición dentro de permisos
definidos y con supervisión humana.

## 2. Problema que resuelve

La empresa necesita conocer sus existencias, controlar las compras
pendientes y anticiparse a la falta de productos.

La revisión manual puede provocar compras duplicadas, reposiciones
tardías y dificultades para explicar diferencias de inventario.

Abastece permitirá consultar la operación y mantener un historial
de las acciones realizadas por usuarios y por el agente.

## 3. Objetivo general

Gestionar el inventario y el abastecimiento de una distribuidora,
manteniendo consistencia en los datos, trazabilidad de las operaciones
y automatización mediante un agente de IA con autonomía limitada.

## 4. Contexto de la primera versión

- Una empresa.
- Una bodega.
- Una moneda: dólares estadounidenses (USD).
- Productos controlados por unidades enteras.
- Proveedores y operaciones de compra simulados.
- Aplicación web de uso interno.
- Sin registro público de usuarios.

## 5. Usuarios y roles

### Administrador

- Gestiona usuarios y asigna roles.
- Administra la configuración del sistema.
- Configura los permisos y límites del agente.

### Gerente

- Consulta inventario, proveedores y compras.
- Aprueba o rechaza solicitudes de compra.
- Aprueba o rechaza ajustes de inventario.
- Supervisa alertas y acciones del agente.

### Encargado de inventario

- Gestiona productos, categorías y proveedores.
- Consulta existencias y movimientos.
- Registra salidas y recepciones de mercancía.
- Registra conteos físicos.
- Solicita ajustes y reposiciones.

Una persona puede tener varios roles. Sus permisos se combinan según
los roles asignados.

## 6. Alcance funcional

### Productos y proveedores

- Gestión de productos y categorías.
- Identificación de productos mediante SKU único.
- Gestión de proveedores.
- Asociación de varios proveedores a un producto.
- Registro de costo, plazo de entrega y cantidad mínima de compra
  por proveedor y producto.

### Inventario

- Consulta de existencias registradas.
- Registro de movimientos de entrada y salida.
- Registro de existencias iniciales mediante movimientos identificados.
- Historial de movimientos por producto.
- Registro de conteos físicos con fecha y responsable.
- Solicitud y aprobación de ajustes por diferencias.
- Prevención de existencias negativas.
- Apertura de inventario por producto, con aprobación del gerente.
- Bloqueo temporal de movimientos por producto durante conteos abiertos.

### Compras y recepción

- Creación de solicitudes de reposición.
- Revisión y aprobación o rechazo de solicitudes.
- Generación de órdenes de compra a partir de solicitudes aprobadas.
- Registro de recepciones parciales o completas.
- Consulta de cantidades pendientes de recibir.
- Cancelación del saldo pendiente de una orden, con motivo registrado.
- Registro de llegadas pendientes de confirmación.
- Confirmación de recepción separada de la llegada física.

### Centro de operaciones

- Consulta de productos con existencias bajas.
- Consulta de solicitudes pendientes de aprobación.
- Seguimiento de órdenes pendientes de recibir.
- Visualización de alertas y actividad del agente.

### Agente de abastecimiento

- Investiga alertas generadas por reglas del sistema.
- Consulta existencias, movimientos y compras pendientes.
- Consulta proveedores y condiciones de suministro.
- Recupera políticas y documentos mediante RAG.
- Propone cantidades y proveedores con una justificación.
- Crea solicitudes de reposición mediante herramientas autorizadas.
- Comprueba si ya existe una reposición abierta para evitar duplicados.
- Solicita intervención humana cuando no puede continuar.
- Conserva el estado del caso para continuar su seguimiento.

### Auditoría

- Identificación del usuario o agente que realiza cada operación.
- Registro de fecha, acción y resultado.
- Seguimiento de herramientas ejecutadas por el agente.
- Registro de aprobaciones, rechazos y errores relevantes.

## 7. Conceptos de inventario

### Existencia registrada

Cantidad que el sistema indica que hay en la bodega, calculada
a partir de los movimientos registrados.

### Existencia física

Cantidad observada durante un conteo físico.

Cada conteo conserva su fecha, responsable y la existencia registrada
con la que se comparó. El último conteo no representa necesariamente
la cantidad física actual si posteriormente hubo movimientos.

Registrar un conteo no modifica automáticamente las existencias.

### Cantidad pendiente de recibir

Cantidad incluida en órdenes de compra emitidas que todavía no se
ha recibido ni cancelado.

Las solicitudes sin convertir en orden no cuentan como mercancía
pendiente de recibir.

## 8. Límites del agente

El agente utiliza una identidad propia y herramientas con permisos
controlados por la aplicación.

En la primera versión puede consultar información, generar alertas
y crear solicitudes de reposición.

No puede:

- Aprobar sus propias solicitudes.
- Emitir órdenes de compra.
- Aprobar ajustes de inventario.
- Confirmar recepciones de mercancía.
- Modificar usuarios, roles o sus propios permisos.
- Ejecutar SQL arbitrario.

Sus acciones deben cumplir las mismas reglas de negocio que las
operaciones realizadas por personas.

## 9. Fuera del alcance inicial

- Varias empresas o bodegas.
- Transferencias entre bodegas.
- Punto de venta y pedidos de clientes.
- Contabilidad, nómina y facturación fiscal.
- Pagos reales y comunicación automática con proveedores externos.
- Devoluciones a proveedores.
- Control por lotes, números de serie o fechas de vencimiento.
- Pronósticos avanzados de demanda.
- Coordinación de múltiples agentes.
- Pronósticos de demanda y estimación de días de cobertura.
- Restricciones de compra por múltiplos de empaque.
- Correcciones de recepciones confirmadas.
- Importación masiva de inventario inicial.

Las salidas por ventas se registrarán como movimientos de inventario,
sin implementar un sistema completo de ventas.

## 10. Escenario principal de demostración

1. El encargado registra una salida de productos.
2. El sistema detecta existencias bajas y genera una alerta.
3. El agente consulta el inventario y las reposiciones abiertas.
4. Consulta proveedores y políticas de abastecimiento.
5. Crea una solicitud de reposición justificada.
6. El gerente revisa y aprueba la solicitud.
7. Se genera y emite una orden de compra simulada.
8. El encargado registra la recepción de la mercancía.
9. El sistema actualiza existencias y cantidades pendientes.
10. El historial permite consultar las acciones de cada participante.

## 11. Criterio de éxito de la primera versión

Completar el escenario principal desde el registro de la salida
hasta la recepción de la compra, con permisos, validaciones,
consistencia de inventario y trazabilidad.

La operación manual de inventario y compras debe seguir disponible
aunque el modelo de IA no esté disponible.
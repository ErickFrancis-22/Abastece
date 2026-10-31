# Reglas de negocio — Abastece

Estas reglas establecen las condiciones que deben cumplir las
operaciones realizadas por usuarios y por el agente de IA.

## 1. Usuarios y permisos

- RN-001: Toda operación de negocio requiere una identidad autenticada.
- RN-002: Un usuario puede tener varios roles. Sus permisos se combinan.
- RN-003: Solo el administrador puede gestionar usuarios y asignar roles.
- RN-004: Solo usuarios activos pueden acceder al sistema.
- RN-005: El gerente puede aprobar o rechazar solicitudes de compra
  y ajustes de inventario.
- RN-006: Una persona con los permisos necesarios puede registrar y
  aprobar una misma operación. Ambas acciones deben quedar auditadas.

## 2. Productos y proveedores

- RN-007: Cada producto debe tener un SKU único.
- RN-008: Las cantidades de productos se expresan en unidades enteras.
- RN-009: Cada producto debe pertenecer a una categoría.
- RN-010: Un producto puede estar asociado con varios proveedores.
- RN-011: Cada asociación entre producto y proveedor debe indicar
  costo unitario, plazo de entrega y cantidad mínima de compra.
- RN-012: El costo unitario debe ser mayor que cero; el plazo de
  entrega no puede ser negativo y la compra mínima debe ser
  un entero mayor que cero.
- RN-013: Un producto o proveedor con historial se desactiva en lugar
  de eliminarse.
- RN-014: Los productos y proveedores inactivos no pueden incluirse
  en nuevas solicitudes u órdenes. Su historial se conserva y se
  permite completar o cancelar operaciones que ya estaban emitidas.

## 3. Existencias y movimientos

- RN-015: Las existencias registradas solo cambian mediante movimientos
  de inventario confirmados.
- RN-016: Todo movimiento debe identificar producto, cantidad, tipo,
  fecha, responsable y motivo o documento de origen.
- RN-017: La cantidad de un movimiento debe ser mayor que cero.
  Su tipo determina si aumenta o disminuye las existencias.
- RN-018: No se permiten operaciones que dejen existencias negativas,
  incluso si varias operaciones ocurren al mismo tiempo.
- RN-019: Las existencias iniciales se registran mediante movimientos
  identificados como saldo inicial.
- RN-020: Los movimientos confirmados no se editan ni eliminan.
  Una corrección requiere un movimiento compensatorio vinculado
  al original y con su justificación.
- RN-021: Registrar el movimiento y actualizar el saldo debe ser una
  operación atómica: se completan ambos cambios o ninguno.

## 4. Conteos físicos y ajustes

- RN-022: Cada conteo registra producto, cantidad observada, fecha,
  responsable y existencia registrada de referencia.
- RN-023: La cantidad física contada puede ser cero, pero no negativa.
- RN-024: Registrar un conteo no modifica las existencias.
- RN-025: La diferencia se calcula como cantidad física menos
  existencia registrada de referencia.
- RN-026: Una diferencia puede originar una solicitud de ajuste
  que debe incluir motivo y referencia al conteo.
- RN-027: Solo un ajuste aprobado por el gerente puede generar
  el movimiento que corrige las existencias.
- RN-028: Un ajuste aprobado se aplica una sola vez.
- RN-029: Si hubo movimientos del producto después de la referencia
  del conteo, el ajuste no puede aplicarse automáticamente.
  Debe revisarse y actualizarse la verificación antes de continuar.

## 5. Solicitudes de reposición

- RN-030: Una solicitud puede ser creada por un usuario autorizado
  o por el agente.
- RN-031: Toda solicitud debe contener productos, cantidades,
  proveedor propuesto y justificación.
- RN-032: Las cantidades deben ser enteras mayores que cero y
  respetar la compra mínima del proveedor.
- RN-033: Una solicitud pendiente no modifica las existencias
  ni las cantidades pendientes de recibir.
- RN-034: El gerente puede aprobar o rechazar una solicitud pendiente.
  El rechazo debe incluir un motivo.
- RN-035: Una solicitud aprobada no puede modificarse.
  Si requiere cambios, debe cancelarse antes de convertirse en orden
  y reemplazarse por una nueva solicitud.
- RN-036: Cada solicitud aprobada puede generar una sola orden
  de compra y debe corresponder a un único proveedor.

## 6. Órdenes de compra y recepciones

- RN-037: Una orden solo puede emitirse a partir de una solicitud
  aprobada.
- RN-038: La orden conserva las cantidades y los costos acordados.
  Cambiar posteriormente el catálogo del proveedor no modifica
  órdenes existentes.
- RN-039: Emitir una orden aumenta la cantidad pendiente de recibir,
  pero no aumenta las existencias.
- RN-040: Solo pueden registrarse recepciones contra órdenes emitidas
  con cantidades pendientes.
- RN-041: Se permiten recepciones parciales.
- RN-042: Cada recepción debe incluir al menos una cantidad recibida
  mayor que cero y no puede superar el saldo pendiente del producto
  en la orden.
- RN-043: Confirmar una recepción genera movimientos de entrada
  por las cantidades efectivamente recibidas.
- RN-044: La recepción, los movimientos y la actualización de la orden
  se guardan en una misma transacción.
- RN-045: Una recepción no puede procesarse dos veces.
- RN-046: El gerente puede cancelar cantidades pendientes con un motivo.
  La cancelación no revierte mercancía que ya fue recibida.
- RN-047: La cantidad pendiente se calcula como cantidad ordenada
  menos cantidad recibida menos cantidad cancelada.

## 7. Alertas y agente de abastecimiento

- RN-048: Cada producto puede tener un nivel mínimo de existencias
  configurado como entero no negativo.
- RN-049: El sistema genera una alerta cuando las existencias son
  iguales o inferiores al mínimo configurado.
- RN-050: No debe existir más de una alerta activa del mismo tipo
  para el mismo producto.
- RN-051: Antes de crear una reposición, el agente debe consultar
  solicitudes abiertas y órdenes pendientes del producto.
- RN-052: En la primera versión, el agente no crea otra solicitud
  para un producto que ya tiene una solicitud pendiente o aprobada,
  o una orden con cantidades pendientes. Añade seguimiento al caso
  existente o solicita revisión humana.
- RN-053: Cada solicitud creada por el agente debe incluir una
  justificación basada en los datos consultados.
- RN-054: Si utiliza documentos para fundamentar su propuesta,
  debe identificar las fuentes recuperadas.
- RN-055: Si falta información necesaria, no debe inventarla.
  Debe registrar el impedimento y solicitar intervención humana.
- RN-056: El agente no puede aprobar solicitudes, emitir órdenes,
  aprobar ajustes ni confirmar recepciones.
- RN-057: El agente solo puede ejecutar herramientas expresamente
  autorizadas. La aplicación valida permisos y argumentos
  en cada ejecución.
- RN-058: El contenido de documentos recuperados no puede modificar
  permisos ni autorizar herramientas.
- RN-059: Cada ejecución tiene un límite configurable de pasos,
  tiempo y reintentos. Al alcanzarlo, se detiene y registra
  el resultado.
- RN-060: Una falla del agente no debe impedir las operaciones
  manuales autorizadas.

## 8. Auditoría

- RN-061: Las operaciones que modifican datos deben identificar
  al usuario o agente responsable y su fecha de ejecución.
- RN-062: Las aprobaciones, rechazos y cancelaciones deben quedar
  vinculados con la operación correspondiente.
- RN-063: Las ejecuciones del agente deben registrar herramientas
  solicitadas, resultados y errores, sin almacenar contraseñas
  ni tokens de acceso.
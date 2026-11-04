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
- RN-019: Las existencias iniciales mayores que cero se registran
  mediante un movimiento de saldo inicial. Una apertura aprobada
  con cero unidades habilita el inventario sin generar movimiento.
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
- RN-026: Una solicitud de ajuste puede originarse en una diferencia
  de conteo físico o en un error de registro. Debe incluir un motivo
  y la referencia al conteo o movimiento que la justifica.
- RN-027: Solo un ajuste aprobado por el gerente puede generar
  el movimiento que corrige las existencias.
- RN-028: Un ajuste aprobado se aplica una sola vez.
- RN-029: Si hubo movimientos del producto después de la referencia
  del conteo, el ajuste no puede aplicarse automáticamente.
  Debe revisarse y actualizarse la verificación antes de continuar.

## 5. Solicitudes de reposición

- RN-030: Una solicitud puede ser creada por un usuario autorizado
  o por el agente.
- RN-031: Toda solicitud debe indicar productos, necesidad estimada,
  cantidad propuesta para comprar, proveedor propuesto, costo estimado
  y justificación. Debe señalar cualquier excedente sobre la necesidad.
- RN-032: La necesidad estimada y la cantidad propuesta deben ser
  enteros mayores que cero. La cantidad propuesta para comprar debe
  respetar el mínimo del proveedor; la necesidad estimada puede ser menor.
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
- RN-044: Confirmar una recepción, generar sus movimientos y actualizar
  existencias, cantidades recibidas y estado de la orden constituye
  una misma transacción. Registrar únicamente la llegada no produce
  esos efectos.
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
- RN-064: Crear una solicitud de reposición no autoriza una compra,
  no emite una orden y no modifica las existencias.
- RN-065: El agente no debe aumentar silenciosamente una cantidad para
  cumplir el mínimo del proveedor. Debe mostrar el excedente, su costo
  y la justificación para que el gerente evalúe la propuesta.
- RN-066: El detalle de una solicitud conserva el costo unitario,
  plazo de entrega y cantidad mínima utilizados al preparar la propuesta.
  Los cambios posteriores del catálogo no sobrescriben esos valores.
- RN-067: Antes de emitir una orden, el sistema compara las condiciones
  actuales registradas con las de la solicitud aprobada. Si difieren,
  bloquea la emisión y exige una nueva revisión.
- RN-068: Una solicitud aprobada que requiere cambios se conserva
  como antecedente y se cancela antes de emitir una orden. Se crea
  una nueva solicitud vinculada como reemplazo, que requiere aprobación.
- RN-069: La orden conserva las condiciones acordadas al emitirla.
  Los cambios posteriores del catálogo no modifican la orden.
- RN-070: Los estados de una solicitud son Pendiente, Aprobada,
  Rechazada, Cancelada y Convertida en orden.
- RN-071: Una solicitud se crea en estado Pendiente. Su contenido
  no se edita después de presentarla; cualquier cambio requiere
  cancelarla y crear una nueva.
- RN-072: Solo el gerente puede aprobar o rechazar una solicitud
  pendiente. Cada decisión queda asociada a esa solicitud concreta
  y no se transfiere a sus reemplazos.
- RN-073: Una solicitud pendiente o aprobada puede cancelarse,
  antes de generar una orden, por un usuario con rol de encargado
  de inventario o gerente. Debe registrarse el motivo.
- RN-074: El agente no puede cancelar solicitudes. Puede preparar
  una propuesta de reemplazo. Si la anterior está pendiente o aprobada,
  un usuario autorizado debe cancelarla antes de presentar el reemplazo.
  Si está rechazada, se conserva en ese estado y se requiere la revisión
  humana prevista para continuar buscando alternativas.
- RN-075: Una solicitud pasa a Convertida en orden únicamente
  cuando su orden se emite correctamente. Ambos cambios deben
  completarse en una misma transacción.
- RN-076: Las solicitudes rechazadas, canceladas o convertidas
  en orden no pueden reabrirse.
- RN-077: En la primera versión, generar y emitir una orden constituye
  una sola operación, autorizada al gerente y basada en una solicitud
  aprobada cuyas condiciones hayan sido verificadas.
- RN-078: Los estados de una orden son Emitida, Parcialmente recibida,
  Completada, Cancelada y Cerrada con recepción parcial.
- RN-079: Una orden puede tener varias recepciones. Cada recepción
  pertenece a una sola orden y conserva su fecha y responsable.
- RN-080: Una orden queda Completada cuando toda la cantidad ordenada
  ha sido recibida, sin cantidades canceladas.
- RN-081: Una orden queda Cancelada si se cancela toda su cantidad
  sin haber recibido mercancía.
- RN-082: Una orden queda Cerrada con recepción parcial cuando
  tiene mercancía recibida y cantidades canceladas, sin saldo pendiente.
- RN-083: Una orden completada, cancelada o cerrada con recepción
  parcial no admite nuevas recepciones.
- RN-084: Cada operación de registro de recepción debe incluir una
  clave de idempotencia única, que se reutiliza en sus reintentos.
- RN-085: Si una recepción ya fue procesada con esa clave y el mismo
  contenido, el sistema devuelve el resultado existente sin crear
  otra recepción ni nuevos movimientos.
- RN-086: Si una clave ya utilizada se presenta con contenido diferente,
  el sistema rechaza la operación por conflicto.
- RN-087: La unicidad de la clave debe protegerse en la base de datos,
  incluso cuando las peticiones llegan simultáneamente.
- RN-088: Si una recepción supera la cantidad pendiente actual de
  cualquier producto, el sistema rechaza la operación completa
  e informa el producto y su cantidad máxima permitida.
- RN-089: El sistema no reduce automáticamente las cantidades
  declaradas por el encargado para ajustarlas al saldo pendiente.
- RN-090: La validación y actualización de las cantidades pendientes
  deben protegerse contra operaciones concurrentes.
- RN-091: Un intento de confirmación rechazado no genera movimientos
  ni modifica existencias o cantidades pendientes. La recepción
  permanece pendiente y puede volver a confirmarse si el impedimento
  desaparece. Si deben corregirse sus cantidades, se cancela y se
  registra otra recepción con una nueva clave de idempotencia.
- RN-092: Una salida debe contener al menos un producto y cantidades
  enteras mayores que cero. Un producto no puede repetirse en el detalle.
- RN-093: Una salida solo puede consumir existencias registradas.
  Las cantidades pendientes de recibir no están disponibles para salir.
- RN-094: Si cualquier producto tiene existencias insuficientes,
  se rechaza la salida completa y se informa la cantidad disponible.
  El sistema no reduce automáticamente las cantidades solicitadas.
- RN-095: La salida, sus detalles, los movimientos y la actualización
  de saldos se confirman en una misma transacción, con protección
  frente a operaciones concurrentes.
- RN-096: Las salidas utilizan una clave de idempotencia para evitar
  registros duplicados. Un reintento con la misma clave y contenido
  devuelve el resultado existente; con contenido distinto se rechaza.
- RN-097: Toda salida requiere un motivo y puede incluir una referencia
  externa, como el número de un comprobante de venta.
- RN-098: Una corrección de un movimiento confirmado requiere una
  solicitud de ajuste y aprobación del gerente. El registro original
  se conserva sin modificaciones.
- RN-099: Al aprobar la corrección, el sistema genera un movimiento
  compensatorio vinculado al original y al ajuste autorizado.
  La aprobación y su aplicación se completan en una misma transacción.
- RN-100: La corrección debe considerar compensaciones anteriores
  para impedir que se reviertan más unidades de las registradas
  en el movimiento original.
- RN-101: En la primera versión, la corrección directa de errores
  de cantidad se limita a salidas de inventario. Las correcciones
  de recepciones y sus efectos sobre órdenes de compra quedan
  fuera del alcance inicial.
- RN-102: Los estados de un ajuste son Pendiente, Aplicado,
  Rechazado y Cancelado. Se crea en estado Pendiente.
- RN-103: Cada ajuste corresponde a un producto y debe referenciar
  exactamente un origen: un conteo físico o un movimiento por corregir.
- RN-104: La diferencia solicitada no puede ser cero.
- RN-105: Aprobar un ajuste y aplicarlo constituye una sola transacción.
  Si falla alguna validación o escritura, no se confirma ningún cambio
  y el ajuste permanece pendiente.
- RN-106: Antes de aplicar un ajuste se verifican nuevamente sus
  condiciones, las correcciones anteriores y las existencias actuales.
  La aprobación del gerente no permite omitir estas validaciones.
- RN-107: Los ajustes aplicados, rechazados o cancelados conservan
  su historial y no pueden editarse ni reabrirse.
- RN-108: Los estados de un conteo físico son Abierto, Confirmado
  y Cancelado.
- RN-109: Solo puede existir un conteo abierto por producto.
  Al abrirlo, el sistema guarda la existencia registrada de referencia.
- RN-110: Mientras exista un conteo abierto, se bloquean las operaciones
  que modifican existencias del producto. Se permite registrar
  llegadas pendientes de confirmación, sin generar movimientos.
  El control debe protegerse frente a operaciones concurrentes.
- RN-111: Confirmar un conteo guarda la cantidad observada,
  calcula la diferencia y libera el bloqueo en una misma transacción.
  No modifica las existencias.
- RN-112: El encargado que abrió el conteo o un gerente puede
  cancelarlo. La cancelación requiere un motivo, libera el bloqueo
  y conserva el historial.
- RN-113: Un conteo confirmado o cancelado no puede editarse
  ni reabrirse. Una nueva verificación requiere otro conteo.
- RN-114: Las recepciones tienen estados Pendiente de confirmación,
  Confirmada y Cancelada. Registrar una llegada no modifica existencias
  ni reduce las cantidades pendientes de recibir de la orden.
- RN-115: La mercancía de una recepción pendiente debe mantenerse
  físicamente separada de las existencias disponibles y de cualquier
  producto que se esté contando.
- RN-116: Confirmar una recepción requiere verificar sus cantidades,
  el saldo pendiente actual de la orden y la ausencia de conteos
  abiertos para sus productos. Solo entonces se generan movimientos
  y se actualiza la orden, en una misma transacción.
- RN-117: Una recepción pendiente no garantiza ni reserva el saldo
  pendiente de la orden. Sus condiciones se validan nuevamente
  al confirmarla.
- RN-118: Una recepción pendiente puede cancelarse por un encargado
  o gerente, indicando el motivo y sin afectar existencias.
  Una recepción confirmada no puede cancelarse mediante este flujo.
- RN-119: Cada producto comienza con saldo cero y estado de inventario
  Pendiente de apertura. No admite movimientos operativos hasta
  completar su apertura.
- RN-120: El encargado solicita la apertura indicando la cantidad física
  inicial, fecha del conteo y justificación. La cantidad debe ser
  un entero no negativo.
- RN-121: Solo el gerente puede aprobar o rechazar una apertura.
  Antes de aprobar debe verificarse que el producto siga pendiente
  de apertura y no tenga movimientos.
- RN-122: Aprobar la apertura, registrar el saldo inicial y habilitar
  el inventario del producto constituye una sola transacción.
  Si la cantidad es cero, se conserva la aprobación sin generar
  un movimiento de cantidad cero.
- RN-123: Cada producto puede completar su apertura una sola vez.
  Las modificaciones posteriores requieren operaciones de inventario
  autorizadas.
- RN-124: Una apertura pendiente no puede ser aprobada por el agente.
  La mercancía debe permanecer sin movimientos físicos durante
  su conteo y hasta completar o cancelar el proceso.
- RN-125: Cada producto tiene un nivel mínimo y un nivel objetivo.
  El mínimo debe ser no negativo y el objetivo debe ser mayor
  que el mínimo.
- RN-126: Para productos con inventario habilitado, se genera una alerta
  cuando la existencia registrada es igual o inferior al nivel mínimo.
- RN-127: Cuando no existen reposiciones abiertas, la necesidad estimada
  se calcula como el nivel objetivo menos la existencia registrada,
  con un límite inferior de cero.
- RN-128: La cantidad propuesta debe cubrir la necesidad estimada
  y respetar la compra mínima del proveedor. Cualquier excedente
  debe mostrarse con su justificación y costo.
- RN-129: La compra mínima no implica que la cantidad deba ser
  un múltiplo de ese valor. Las restricciones por múltiplos de empaque
  quedan fuera del alcance inicial.
- RN-130: Los cálculos de cantidades y costos se realizan mediante
  reglas de la aplicación. El agente puede consultar sus resultados,
  pero no omitir ni modificar esas reglas.
- RN-131: La primera versión utiliza niveles configurados de reposición.
  No implementa pronósticos de demanda ni cálculos de días de cobertura.
- RN-132: Una asignación de rol se identifica por la combinación
  de usuario y rol. No se permiten asignaciones duplicadas.
- RN-133: Retirar un rol elimina la asignación, pero conserva
  al usuario y su historial. El cambio debe quedar auditado.
- RN-134: Las operaciones protegidas verifican que el usuario
  esté activo y tenga los permisos vigentes requeridos.
  Un JWT anterior no conserva permisos que ya fueron retirados.
- RN-135: El rol Administrador no concede automáticamente permisos
  de Gerente ni de Encargado de inventario.
- RN-136: Un administrador no puede modificar sus propias asignaciones
  de roles mediante la API. Otro administrador debe realizar el cambio.
- RN-137: No se permite desactivar al último administrador activo
  ni retirar su rol de Administrador. Esta validación debe protegerse
  frente a operaciones concurrentes.
- RN-138: Toda solicitud identifica a su creador mediante una identidad
  de actor. La revisión identifica al usuario humano que tomó la decisión.
- RN-139: Los permisos se verifican al ejecutar la acción. Retirar
  posteriormente un rol no elimina ni invalida decisiones anteriores
  realizadas con autorización.
- RN-140: Cada usuario y cada agente tienen una identidad de actor
  propia y exclusiva. El tipo de actor debe corresponder a su perfil.
- RN-141: Las identidades con historial no se eliminan físicamente.
  Los usuarios se desactivan y los agentes se deshabilitan.
- RN-142: Deshabilitar un agente impide iniciar nuevas ejecuciones
  y autorizar nuevas llamadas a herramientas. No elimina su historial
  ni cancela automáticamente las solicitudes que creó.
- RN-143: Antes de ejecutar cada herramienta se verifica que el agente
  continúe habilitado y autorizado. Las operaciones ya confirmadas
  conservan sus efectos.
- RN-144: Los casos conservan su estado entre ejecuciones. Esperar
  una aprobación humana no requiere mantener una ejecución activa.
- RN-145: Cada llamada a herramienta se vincula con una ejecución
  y registra su resultado, sin exponer secretos.
- RN-146: La trazabilidad del agente conserva acciones, fuentes
  y justificaciones verificables; no requiere almacenar razonamientos
  internos del modelo.
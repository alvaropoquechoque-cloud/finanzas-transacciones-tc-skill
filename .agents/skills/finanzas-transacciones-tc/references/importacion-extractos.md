# Importación de extractos bancarios

## Objetivo

Definir cómo convertir extractos bancarios y estados de cuenta en movimientos confiables dentro de `Transacciones`.

La importación debe preservar trazabilidad, evitar duplicados y permitir que las vistas dependientes se actualicen desde una sola fuente de verdad.

## Fuente de verdad

`Transacciones` es la fuente de verdad operativa.

Un extracto bancario:
- confirma movimientos realizados;
- ayuda a completar fecha, monto, banco/cuenta y conciliación;
- no debe alimentar directamente `CxC Mensual`, `CxP Mensual`, `Real S&A`, `Operative incomes`, `Bancos`, `Runway Mensual` ni `Dashboard`.

Las vistas derivadas deben actualizarse desde `Transacciones`.

## Flujo estándar

1. Recibir el extracto.
2. Identificar banco/cuenta y periodo.
3. Leer todos los movimientos.
4. Normalizar fechas, moneda, descripción y monto.
5. Buscar duplicados en `Transacciones`.
6. Identificar si cada movimiento es:
   - ingreso;
   - egreso;
   - transferencia interna;
   - comisión bancaria;
   - conversión de moneda;
   - pago/cobro de una obligación existente.
7. Registrar únicamente movimientos que no existan.
8. Completar o actualizar transacciones existentes cuando el extracto confirme su pago/cobro.
9. Aplicar categorización.
10. Aplicar TC según las reglas vigentes.
11. Conciliar contra `Bancos`.
12. Verificar vistas dependientes.

## Regla de no duplicación

Antes de insertar un movimiento, buscar coincidencias utilizando el mayor número posible de estos atributos:

- fecha;
- banco/cuenta;
- moneda;
- monto;
- descripción;
- contraparte;
- tipo;
- cuenta origen;
- cuenta destino.

No asumir que una descripción diferente implica una transacción diferente.

Si un movimiento bancario corresponde a una CxC o CxP ya registrada como `Pendiente`, no crear una segunda transacción para liquidarla.

Se debe actualizar la transacción existente:
- estado;
- fecha de pago/cobro;
- conciliación;
- cuenta bancaria involucrada;
- datos faltantes que el extracto permita confirmar.

## Una fila por movimiento bancario

Como regla general, cada movimiento independiente del extracto debe conservarse como una fila independiente en `Transacciones`.

No agrupar varios movimientos solo porque:
- pertenecen al mismo proveedor;
- ocurrieron el mismo día;
- corresponden al mismo concepto;
- forman parte de una misma transferencia mayor.

Esto permite conciliar exactamente contra el banco.

Ejemplo:

Si una plataforma entrega:
- USD 3,996
- USD 2

como dos movimientos bancarios separados, deben mantenerse como dos movimientos separados si así aparecen en el extracto.

## Transferencias internas

Una transferencia entre cuentas propias no representa ingreso ni gasto.

Debe registrarse como:

- Tipo: `Transferencia interna`
- Categoría: `Transferencias internas`

Además, completar cuando sea posible:
- `Cuenta origen`
- `Cuenta destino`
- `Detalle transferencia`

La dirección de la transferencia es obligatoria para que la conciliación bancaria pueda sumar correctamente entradas y salidas por cuenta.

### Ejemplo

Si Meru envía fondos que posteriormente ingresan a Banco Sol mediante un intermediario como RemotePay:

Movimiento de salida:
- Cuenta origen: Meru
- Cuenta destino: Banco Sol o cuenta intermedia identificada
- Tipo: Transferencia interna

Movimiento de entrada:
- Banco/cuenta: Banco Sol
- Tipo: Transferencia interna
- Cuenta origen: Meru o intermediario confirmado
- Cuenta destino: Banco Sol

No reconocer el traslado como ingreso operativo.

## Intermediarios de pago

Cuando el extracto muestre un intermediario como:
- RemotePay;
- procesador de pagos;
- billetera;
- banco corresponsal;

no asumir automáticamente que el intermediario es el cliente o proveedor económico.

Se debe separar:

1. contraparte bancaria;
2. contraparte económica;
3. cuenta origen/destino;
4. concepto real.

Ejemplo:

Una entrada de `RemotePay Solutions` puede ser una transferencia proveniente de Meru y no un ingreso de RemotePay.

## Comisiones bancarias

Una comisión cobrada por el banco o plataforma debe registrarse como un movimiento separado cuando el extracto la muestre de forma independiente.

Categoría habitual:
`Bank fees`

No modificar artificialmente el importe principal para hacer coincidir el neto recibido.

Ejemplo:

Si se enviaron USD 4,000 y el banco registra:
- USD 3,998 recibidos;
- USD 2 de comisión;

preservar ambos componentes según el detalle bancario disponible.

## Conversión de moneda

No convertir manualmente el monto original del extracto.

Registrar:
- moneda original;
- monto original;
- TC correspondiente;
- monto USD calculado.

Reglas conocidas:
- USD → TC 1
- SOL → TC 0.28
- BOB → TC oficial BCB de la fecha
- otras monedas → TC manual cuando corresponda

Para BOB, si la fecha cae en fin de semana o feriado, usar el último TC oficial disponible anterior o igual a la fecha.

## Fecha del movimiento

Usar la fecha efectiva mostrada por el extracto.

No sustituirla por:
- fecha de generación del estado de cuenta;
- fecha de factura;
- fecha de vencimiento.

Si una factura pendiente se liquida con el movimiento:

- conservar la fecha original de factura/transacción;
- actualizar `Fecha pago / cobro` con la fecha bancaria real.

Esto permite mantener simultáneamente:
- devengamiento;
- vencimiento;
- fecha real de caja.

## Pagos de CxP

Si el extracto confirma el pago de una obligación ya existente:

1. localizar la transacción pendiente;
2. validar monto y contraparte;
3. completar cuenta bancaria;
4. completar `Fecha pago / cobro`;
5. cambiar el estado a `Pagado/Cobrado`;
6. marcar conciliación según corresponda.

No crear un nuevo egreso si la obligación ya estaba registrada.

## Cobros de CxC

Si el extracto confirma el cobro de una cuenta ya existente:

1. localizar la transacción pendiente;
2. validar monto y contraparte;
3. completar banco/cuenta;
4. completar `Fecha pago / cobro`;
5. cambiar el estado a `Pagado/Cobrado`;
6. conciliar.

No crear un segundo ingreso para representar el mismo cobro.

## Pagos parciales

Si el pago o cobro es parcial, no marcar automáticamente toda la obligación como liquidada.

Primero determinar:
- monto original;
- monto pagado/cobrado;
- saldo pendiente;
- si el modelo permite dividir la obligación en componentes.

No inventar una metodología de partición si no está definida.

## Extractos incompletos

Antes de cerrar un mes, confirmar que el extracto cubre el periodo completo.

Si el archivo empieza o termina fuera del periodo esperado:
- no asumir que no existieron movimientos;
- marcar la conciliación como incompleta;
- identificar qué fechas faltan.

Ejemplo:
un extracto que comienza el 3 de agosto no prueba que el 1 y 2 de agosto no tuvieron movimientos.

## Descripciones bancarias

Preservar información útil de la descripción original.

La descripción normalizada debe permitir identificar:
- contraparte;
- concepto;
- periodo cuando aplique;
- número de factura si está disponible.

No reemplazar toda la descripción bancaria con una etiqueta genérica si se pierde trazabilidad.

## Estados de cuenta de proveedores

Un estado de cuenta de proveedor no es un extracto bancario.

Puede utilizarse para:
- identificar facturas;
- validar monto;
- validar fecha;
- validar proveedor;
- completar CxP.

Pero una factura o estado de cuenta no prueba por sí solo que el pago haya ocurrido.

El pago debe confirmarse mediante:
- extracto;
- comprobante;
- evidencia equivalente.

## Categorización

La importación no debe inventar categorías.

Orden recomendado:

1. buscar regla activa en `Reglas categorización`;
2. usar una categoría válida de `Config`;
3. si no existe coincidencia confiable, usar `Por categorizar`.

Una importación puede completarse aunque la categoría quede temporalmente pendiente.

La conciliación bancaria y la categorización son controles distintos.

## Bancos y cuentas conocidas

La denominación debe mantenerse consistente con los nombres utilizados actualmente en el modelo.

Ejemplos conocidos:
- Banco Sol
- Brex
- Brex Card
- Meru
- Scotiabank
- BCI

Antes de crear un nombre nuevo, revisar si la cuenta ya existe con otra denominación.

## Conciliación posterior a la importación

Después de importar un extracto, validar por banco y mes:

`Saldo final calculado = Saldo inicial + Entradas - Salidas`

Luego:

`Diferencia = Saldo final banco - Saldo final calculado`

La diferencia esperada debe ser cero o estar dentro de la tolerancia documentada.

No insertar movimientos artificiales para hacer que el banco concilie.

## Si existe una diferencia

Investigar en este orden:

1. movimientos faltantes;
2. duplicados;
3. transferencias internas sin origen/destino;
4. movimientos registrados en banco incorrecto;
5. fecha incorrecta;
6. monto o moneda incorrectos;
7. comisión bancaria faltante;
8. tipo de cambio incorrecto;
9. extracto incompleto.

## Verificación posterior

Después de cada importación revisar:

- `Transacciones`
- `Bancos`
- `CxC Mensual`
- `CxP Mensual`
- `Operative incomes`
- `Real S&A`
- `Runway Mensual`
- `Dashboard`

Buscar además:
- `#REF!`
- `#VALUE!`
- `#N/A`
- `#ERROR!`

## Reglas de seguridad

- Leer encabezados actuales antes de escribir.
- No asumir posiciones históricas de columnas.
- No borrar movimientos existentes para reemplazarlos por el extracto.
- No duplicar CxC/CxP al momento de cobrar o pagar.
- No inventar país, responsable, cuenta, contraparte o fecha.
- No modificar la categoría del usuario sin evidencia suficiente.
- No forzar conciliaciones.
- No confundir transferencias internas con ingreso o gasto.
- No tratar una factura como evidencia de pago.
- Si el extracto y el Sheet presentan una inconsistencia, investigar antes de corregir.
- El Google Sheet vivo prevalece sobre snapshots históricos de GitHub.

## Resultado esperado

Una importación se considera terminada cuando:

- todos los movimientos relevantes del periodo están registrados;
- no existen duplicados conocidos;
- las transferencias tienen dirección correcta;
- pagos y cobros existentes fueron vinculados a sus obligaciones;
- el TC fue aplicado correctamente;
- la cuenta bancaria concilia;
- no se introdujeron errores en las vistas dependientes.

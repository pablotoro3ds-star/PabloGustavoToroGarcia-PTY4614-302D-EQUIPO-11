# Acta de reunión — Kuentas · 30-09-2026

> Levantamiento con Claudio (contador) sobre integración con el SII, automatización
> contable y ajustes al sistema. Base para las specs de la **Fase 7** del roadmap.

| | |
|---|---|
| **Fecha** | 30 de septiembre de 2026 |
| **Duración total del audio** | 2 h 37 min |
| **Tramo contable analizado** | 00:00 – 02:13:45 y 02:31:30 – 02:37:00 |
| **Tramo excluido** | 02:13:45 – 02:31:30 (punto de venta, fuera de alcance) |
| **Fuentes** | Transcripción automática del audio + notas de WhatsApp del equipo |

**Participantes**

- **Claudio** — contador, cliente piloto y validador funcional. Es quien define las reglas de negocio.
- **Rodrigo** — dirección técnica y comercial del producto.
- **Gabriel / equipo de desarrollo** — muestra el sistema, revisa el catálogo del API Gateway y registra los acuerdos.

> **Nota sobre la transcripción:** el audio se transcribió con una herramienta automática y tiene
> errores de reconocimiento (nombres, siglas y cifras). Los contenidos de esta acta son una
> reconstrucción de los acuerdos, no citas literales. Las marcas de tiempo son aproximadas.

---

## 1. Resumen ejecutivo

1. El objetivo de fondo es **contabilizar con un botón**: bajar el libro de compras y que el sistema proponga el asiento completo, incluida la cuenta de gasto, usando memoria por RUT, la glosa del documento y el giro del tercero.
2. La integración con el SII se hará a través de **apigateway.cl**, un servicio externo de pago por consumo, en vez de almacenar el RUT y la clave tributaria del contribuyente.
3. El sistema debe nacer **multimoneda**, con dos decimales en el almacenamiento desde el primer día. Es una decisión de arquitectura que no se puede postergar.
4. El **control de folios y la impresión de libros oficiales en PDF** se identificó como diferenciador: ningún competidor lo resuelve.
5. Se acordó un **plan de pruebas con Claudio desde el lunes siguiente**, con acceso al sistema y trabajo en paralelo.
6. La meta comercial es tener el sistema **operativo el 1 de enero de 2027**, con tres meses de desarrollo.

---

## 2. Minuta cronológica

### 00:00 – 06:00 · Contabilización automática de las compras

Es el requerimiento central de la reunión. Hoy, al bajar el libro de compras, se pueden deducir dos
de las tres cuentas del asiento (IVA crédito fiscal y proveedores), pero no la cuenta de gasto o
resultado, porque el SII no la entrega.

Solución acordada, en tres niveles:

1. **Memoria por RUT.** La primera vez el usuario asigna la cuenta (ejemplo: la boleta de luz a "gastos de luz") y el sistema guarda la asociación RUT → cuenta. Desde ahí el asiento se arma solo.
2. **RUT nuevo con glosa.** La IA lee la glosa del documento y propone la cuenta.
3. **RUT nuevo sin glosa.** Se consulta la situación tributaria del tercero, se obtiene el giro o código de actividad económica y la IA busca en el plan de cuentas la cuenta de gasto más parecida.

En todos los casos es una **propuesta**, no una asignación automática: el usuario confirma.

Complemento pedido por Claudio: al terminar la descarga, el sistema debe **listar primero las compras
que quedaron sin cuenta de gasto asignada**. MasTecno las manda por defecto a caja, lo que desordena
la contabilidad y obliga a buscarlas una por una.

### 06:00 – 09:00 · Home y datos de la empresa

- Un botón junto al ícono de inicio que despliegue los **datos de la empresa**: inicio de actividades, socios, capitales y representantes.
- El **cambio de empresa** hoy está abajo a la izquierda; debe subir a las primeras opciones del menú.
- Discusión abierta sobre el estilo del menú: tarjetas o menú lateral.

### 09:00 – 15:20 · Datos del contribuyente y consulta única

Al crear la empresa, MasTecno pide RUT y clave y baja solo el nombre y la dirección. Claudio necesita
además el **representante legal, los socios, el capital enterado y el capital por enterar**.

Regla de ahorro acordada, porque el API Gateway se cobra por consumo: **consultar una vez y guardar en
la base de datos**. Solo se vuelve a consultar cuando el RUT es nuevo o para una actualización
periódica, ya que estos datos solo cambian si el contribuyente modifica su giro o su razón social.

### 15:20 – 18:10 · Documentos emitidos

Se confirma el uso de **documentos emitidos por emisor y período** para generar los asientos de venta.
Queda la pregunta de si los libros deben quedar **idénticos a los de MasTecno** (compras, ventas y
honorarios).

### 18:10 – 23:00 · Multimoneda y decimales

Claudio relata que pidió a MasContable una versión en dólares y le respondieron que no se podía, por
el riesgo de modificar las tablas de todos sus clientes en producción.

Acuerdos:

- El sistema **almacena siempre con dos decimales**, desde el inicio. Hacerlo después implicaría migrar la base con el servicio caído.
- Al crear la empresa se elige la moneda. Si es nacional, los decimales **se ocultan visualmente**, pero siguen existiendo en el dato.
- La aspiración es multimoneda real: dólar, euro, real y peso colombiano, pensando en clientes fuera de Chile.
- Queda por definir el criterio de tipo de cambio: valor en tiempo real contra valor de cierre de mes.

### 23:00 – 29:00 · Honorarios y boletas de terceros

- **Boletas de honorarios recibidas:** se usan. El PDF **no**: si se necesita, se descarga del sitio del SII.
- **Boletas de terceros recibidas:** se usan. Son las que emite la empresa por un prestador sin inicio de actividades; la empresa es responsable de declarar la retención.
- **BTE emitidas, resumen por emisor y año:** se usa para el cruce anual y para detectar boletas anuladas.
- **Emitir o anular boletas desde el sistema:** descartado en esta etapa; se hace en el sitio del SII.

### 29:00 – 38:00 · Folios, timbraje y libros impresos

Claudio lo describe como el mayor diferenciador: "ningún sistema lo tiene".

El flujo pedido:

1. Consultar en el SII cuántos folios (hojas legales) tiene **autorizados y disponibles** la empresa.
2. Al imprimir, asignar folios en este orden: **libro diario, libro mayor, inventario y balance, balance de 8 columnas**.
3. Mantener un **cuadro de control** con libro, folio desde, folio hasta y año.
4. Generar el **PDF** del libro foliado.

Al año siguiente el sistema debe continuar la numeración desde el último folio usado.

### 38:00 – 45:00 · Registro de compras

- Se usa el **detalle de compras**; el **resumen lo calcula el sistema** a partir del detalle.
- Para volúmenes grandes hay que implementar el flujo asíncrono de tres pasos: **solicitar la descarga, consultar el estado y obtener el detalle**. Claudio cuenta que MasTecno corta alrededor de los 700 registros y obliga a esperar entre 15 y 30 minutos, porque el propio SII limita el consumo.
- Se descarta la **asignación de tipo de transacción**, porque el código se asigna una sola vez y no cambia.

### 45:00 – 53:00 · Registro de ventas

- Se usa el **detalle de ventas**; el resumen también lo calcula el sistema.
- Se descarta la **asignación de resumen de ventas por documento**.
- El cruce entre lo vendido en un punto de venta y lo registrado en el SII se deja para más adelante: es un módulo que se suma después, cuando el cliente tenga su propio sistema de ventas.

### 53:00 – 58:00 · Contribuciones y bienes raíces

Claudio necesita que el sistema **alerte el vencimiento de las contribuciones**, que se pagan en cuatro
cuotas al año, y que genere el comprobante contable del pago. En el catálogo revisado **no se encontró
un recurso de pago de contribuciones**: solo datos de propiedades y certificados. Queda pendiente.

### 58:00 – 01:02:20 · Confirmación de la consulta de terceros

Se vuelve al recurso **situación tributaria de terceros**, dentro de consultas públicas de
contribuyentes. Entrega RUT, razón social, inicio de actividades y actividad económica: es la pieza que
alimenta la propuesta de cuenta descrita en el primer bloque.

### 01:02:20 – 01:08:00 · Plan de pruebas y módulos pendientes

- Claudio probará el sistema ingresando comprobantes, para verificar que la clonación de MasContable funcione en la práctica. Requisito previo: el **plan de cuentas cargado**.
- Se acuerda **acceso con dos usuarios en paralelo**: uno prueba y anota, otro corrige.
- El **módulo de remuneraciones** debe generar el libro que se sube a la Dirección del Trabajo, el comprobante contable y el indicador mensual de impuesto de segunda categoría.
- **Certificación ante el SII:** se entiende que solo aplica al módulo de renta cuando se importan datos. Alternativa intermedia: generar el archivo con el formato exigido para subirlo manualmente.

### 01:08:00 – 01:18:00 · Conciliación bancaria y pagos

El problema: la cartola no dice qué factura se está pagando, y los sistemas actuales solo comparan
totales por día en vez de identificar la transacción.

Propuestas conversadas:

- Que el **pago a proveedores se genere desde el sistema**: el usuario elige la factura, el sistema deriva al banco para la autorización y deja el voucher contable creado con el número de factura. El cruce queda resuelto en el origen.
- Pedir a los clientes que indiquen el **número de factura en la glosa de la transferencia**.
- Algunos bancos ofrecen **API** y otros formatos de pago masivo (se menciona el de Santander). Lo ideal sería bajar la cartola a diario.
- Para lo no identificado, usar una **cuenta puente de fondos por rendir** que se va limpiando.

### 01:18:00 – 01:22:30 · Escenario de prueba y home del dueño

Prueba comprometida: **API del SII operativa, descarga de libros, bajar las facturas de enero de un
cliente, generar el asiento contable, registrar el pago de contribuciones e impuestos y ver la
propuesta de Formulario 29 en el home**.

Sobre el home: debe adaptarse al perfil de quien mira. El dueño quiere ver **las ventas del día en un
gráfico de línea**, luego las compras y después sus obligaciones de corto plazo (F29, previsional,
sueldos). Debe verse bien en celular.

### 01:22:30 – 01:45:00 · Producto, competencia e infraestructura

- Nombre del producto **sin definir**.
- Competencia de referencia: **Came** (software contable robusto y caro) y **Buk** para remuneraciones. Se menciona la experiencia previa con SAP y con un ERP brasileño como contraste entre rigidez y personalización.
- Se descarta almacenar el RUT y la clave tributaria del contribuyente por el riesgo que implica, y se opta por **apigateway.cl**, de pago por consumo, cuyo catálogo se revisa en la reunión.

### 01:45:00 – 01:52:00 · Otros recursos del catálogo

Se revisan recursos adicionales: **UF y otros indicadores** (con origen en el Banco Central, sin costo),
**impuesto de segunda categoría**, verificación de RUT y situación tributaria de terceros. También se
menciona que el catálogo incluye bienes raíces y vehículos, fuera de alcance por ahora.

### 01:52:00 – 02:00:00 · Migración desde MasContable

El traspaso se hace con un **asiento de apertura**. Claudio advierte que un asiento de apertura por
totales pierde el análisis: hay que traer el **detalle por RUT de clientes y proveedores**, es decir,
qué documentos componen cada saldo. Hay que preparar ese asiento para cada cliente antes del cierre
del año.

### 02:00:00 – 02:13:45 · Metas y estrategia

- Sistema **operativo el 1 de enero de 2027**; tres meses de desarrollo, partiendo del 1 de noviembre.
- La óptica es el **cliente piloto**; el foco real es el sistema contable.
- Canal de venta: **colegio de contadores**, institutos y universidades, apoyados por un profesor con red nacional.
- Se propone una **red de contadores** que retroalimente el producto.

### 02:31:30 – 02:37:00 · Plan de cuentas y cierre

- Las cuentas en **rojo son las intocables** y se marcarán con un **candado**, con opción de habilitarlo y deshabilitarlo.
- La estructura base está amarrada a la **declaración jurada del balance** que se presenta el 30 de junio: el código madre lo define el SII (por ejemplo, 1.000.000 "Disponible") y las subcuentas cuelgan de ese concepto. El sistema suma cada concepto y lo lleva a la línea correspondiente de la declaración.
- Riesgo detectado: el plan de cuentas existe tanto en el sistema contable como en el módulo de gestión; hay que evitar que se editen por separado y queden inconsistentes.
- Se acuerda dar acceso a Claudio cuanto antes y trabajar con una bitácora de avances, pendientes y errores.

---

## 3. Requerimientos derivados

Prioridad: **A** = bloqueante para la prueba con Claudio · **B** = necesario para el 1 de enero · **C** = posterior.

| # | Requerimiento | Prioridad | Spec |
|---|---|---|---|
| R1 | Propuesta automática de cuenta de gasto: memoria RUT → cuenta, lectura de glosa y giro del tercero | A | Nueva (extiende 066 y 069) |
| R2 | Listado de compras descargadas sin cuenta de gasto asignada, antes de contabilizar | A | Nueva |
| R3 | Integración real con el SII vía apigateway.cl, sin almacenar la clave tributaria | A | SPEC-066 |
| R4 | Descarga de compras asíncrona en tres pasos, con control del límite de registros | A | SPEC-066 |
| R5 | Resumen de compras y de ventas calculado por el sistema desde el detalle | A | SPEC-066 |
| R6 | Datos del contribuyente al crear la empresa: representantes, socios, capital enterado y por enterar, con consulta única y caché | A | SPEC-066 |
| R7 | Multimoneda: almacenamiento con dos decimales y moneda por empresa, con ocultamiento visual en moneda nacional | A | Nueva (transversal al modelo de datos) |
| R8 | Plan de cuentas: candado en las cuentas intocables, habilitable y deshabilitable | A | SPEC-002 / 013 |
| R9 | Control de folios: consulta de disponibles en el SII, asignación por libro, cuadro de control y continuidad entre años | B | Nueva (usa el recurso de timbraje) |
| R10 | Impresión de libros oficiales en PDF generado por el sistema, foliados y registrados | B | Nueva (reemplaza el `window.print()` actual) |
| R11 | Propuesta de Formulario 29 visible en el home | B | SPEC-061 |
| R12 | Home por perfil: ventas del día en gráfico de línea, compras y obligaciones de corto plazo, con vista móvil | B | SPEC-061 |
| R13 | Datos de la empresa desplegables desde el home y selector de empresa en las primeras opciones del menú | B | SPEC-060 |
| R14 | Alerta de vencimiento de contribuciones (4 cuotas al año) y comprobante del pago | B | Nueva |
| R15 | Remuneraciones: libro para la Dirección del Trabajo, comprobante contable e indicador mensual de impuesto de segunda categoría | B | SPEC-072 |
| R16 | Corrección monetaria anual e impuesto de segunda categoría anual | B | SPEC-067 / 068 |
| R17 | Comprobantes tipo: crear y recuperar desde la misma pantalla, sin salir del módulo | B | SPEC-018 (mejora de UX) |
| R18 | Mantener la última fecha registrada al ingresar comprobantes | B | SPEC-017 |
| R19 | Reordenamiento y renumeración de comprobantes por fecha, previo a la foliación | B | SPEC-055 |
| R20 | Migración desde MasContable: asiento de apertura con detalle por RUT de clientes y proveedores | B | Nueva |
| R21 | Pago a proveedores desde el sistema, con derivación al banco y voucher con número de factura | C | SPEC-039 / 040 |
| R22 | Conciliación bancaria por identificación de transacción, con cartola diaria por API y cuenta puente para lo no identificado | C | SPEC-040 |
| R23 | Cruce entre el punto de venta del cliente y lo registrado en el SII | C | Nueva |

---

## 4. Recursos del API Gateway

### Se usan

| Recurso | Uso en Kuentas |
|---|---|
| Situación tributaria de terceros (consultas públicas, por RUT) | Giro y actividad económica del proveedor; alimenta la propuesta de cuenta |
| Actividades económicas | Catálogo de giros |
| Contribuyentes datos | Capital enterado y por enterar, y representantes; solo al crear la empresa |
| Documentos emitidos (emisor / período) | Asientos de venta |
| Boletas de honorarios recibidas | Registro de honorarios (sin PDF) |
| Boletas de terceros recibidas | Control de la retención cuando se emite por un prestador sin inicio de actividades |
| BTE emitidas, resumen (emisor / año) | Cruce anual y detección de anulaciones |
| Contribuyentes autorizados a emitir DTE | Validación del emisor |
| Asignación de folios · estado de timbraje | Control de folios disponibles y usados |
| Registro de compras · detalle | Contabilización de compras |
| Solicitar descarga de compras · estado · detalle | Volúmenes sobre el límite de registros |
| Registro de ventas · detalle | Contabilización de ventas |
| UF e indicadores; impuesto de segunda categoría | Corrección monetaria y remuneraciones |

### No se usan por ahora

Emisión y anulación de boletas desde el sistema · descarga de PDF de boletas · asignación de tipo de
transacción de compra · asignación de resumen de ventas por documento · descarga asíncrona de ventas ·
cesión de documentos electrónicos · bienes raíces y vehículos · cargas masivas de contribuyentes ·
código de reemplazo de libros (solo períodos hasta 2017).

---

## 5. Escenario de prueba comprometido

1. API del SII operativa a través del gateway.
2. Descarga de los libros de compras, ventas y honorarios.
3. Bajar las facturas de enero de un cliente real.
4. Generar el asiento contable a partir de esas facturas.
5. Registrar el pago de contribuciones y de impuestos.
6. Ver la propuesta de Formulario 29 en el home.

---

## 6. Acuerdos y tareas

| Tarea | Responsable | Plazo |
|---|---|---|
| Dejar el plan de cuentas cargado y operativo, con el candado en las cuentas intocables | Equipo de desarrollo | Lunes siguiente |
| Dar acceso a Claudio con dos usuarios en paralelo | Equipo de desarrollo | Lunes siguiente |
| Bitácora de avances, pendientes y errores durante las pruebas | Equipo de desarrollo | Durante las pruebas |
| Contratar y conectar apigateway.cl, partiendo por una empresa | Rodrigo | Antes de la prueba |
| Probar el ingreso de comprobantes y reportar errores | Claudio | Semana siguiente |
| Acompañamiento presencial en la oficina | Rodrigo | Miércoles y viernes |
| Preparar el asiento de apertura por cliente para la migración | Claudio | Antes del cierre de año |
| Definir el nombre del producto | Todos | Por definir |

---

## 7. Dudas abiertas

Continúan la numeración de `docs/dudas-validacion-claudio-demo.md`.

| # | Duda | Quién responde |
|---|---|---|
| D19 | ¿Los libros de compras, ventas y honorarios deben quedar idénticos a los de MasTecno? | Claudio |
| D20 | Número de la declaración jurada del balance: en la reunión se dijo 1948 y en las notas 1847 | Claudio |
| D21 | Criterio de tipo de cambio en multimoneda: tiempo real o cierre de mes | Claudio / Rodrigo |
| D22 | ¿Existe un recurso de pago de contribuciones en el gateway o hay que resolverlo por otra vía? | Equipo |
| D23 | Alcance real de la certificación ante el SII para el módulo de renta | Claudio |
| D24 | Código de identificación en la glosa de la transferencia para el cruce de la conciliación | Claudio |
| D25 | Cómo evitar que el plan de cuentas se edite por separado en el sistema contable y en el de gestión | Equipo |
| D26 | Nombre definitivo del producto | Todos |

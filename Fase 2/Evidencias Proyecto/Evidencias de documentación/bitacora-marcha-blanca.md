# Bitácora de marcha blanca — Kuentas

> Registro de las pruebas del contador sobre el sistema en producción, con sus
> hallazgos y la resolución de cada uno. Evidencia de validación funcional con el
> usuario real del producto.

| | |
|---|---|
| **Producto** | Kuentas · plataforma contable SaaS |
| **Ambiente** | Producción · Cloud Run (`southamerica-west1`) |
| **Usuario que prueba** | Claudio — contador, cliente piloto |
| **Equipo de desarrollo** | Pablo Toro y Gabriel Bermar |
| **Inicio del registro** | 23 de septiembre de 2026 |
| **Última actualización** | 22 de octubre de 2026 |

---

## 1. Cómo se registra

Cada vez que el contador usa el sistema se agrega una fila a la tabla de sesiones y,
si aparece un problema, una ficha de hallazgo. La regla es simple: **si no quedó
escrito, no ocurrió**. Un hallazgo se cierra solo cuando el cambio está en `main`,
desplegado, y el contador confirma que el problema no se repite.

**Estados de un hallazgo:** abierto · en análisis · corregido (sin confirmar) · cerrado · descartado.

**Severidad:** bloqueante (impide seguir trabajando) · alta (obliga a un rodeo) ·
media (molesta pero no detiene) · baja (cosmético o sugerencia).

---

## 2. Sesiones de prueba

| # | Fecha | Participantes | Alcance de la sesión | Hallazgos |
|---|---|---|---|---|
| S1 | 23-09-2026 | Claudio, Pablo, Gabriel, Rodrigo | Demostración del sistema completo: recorrido por los 50 módulos tras el cierre de la clonación. | Se levantaron los requerimientos de la etapa siguiente (13 specs) y las dudas D1–D18. |
| S2 | 22-10-2026 | Pablo Toro | Recorrido completo del sistema en producción sobre la empresa de prueba: configuración, maestros, captura, libros, informes y cierre. Se verificó el ciclo contable de punta a punta. | H2, H3, H4, H5 |
| S3 | *(por registrar)* | | Primer ingreso del contador con su cuenta propia: cambio de contraseña, creación de la empresa y carga del plan de cuentas. | |
| S4 | *(por registrar)* | | Descarga de compras y ventas, y contabilización asistida. | |
| S5 | *(por registrar)* | | Cierre de período: Estado de Resultado, balances y propuesta de Formulario 29. | |

---

## 3. Hallazgos

### H1 · Caída al abrir el detalle en el Libro Mayor

| | |
|---|---|
| **Reportado** | Por el cliente, sobre la vista de Libro Mayor |
| **Severidad** | Alta |
| **Estado** | Corregido, sin confirmar por el usuario |

**Descripción.** Al abrir el detalle de una cuenta, la vista se caía en lugar de mostrar
los movimientos.

**Resolución.** Se blindó el render de la vista y se agregó el enlace desde el movimiento
al comprobante de origen (PR #102, integrado el 13-10-2026).

**Pendiente.** No se logró reproducir el error original, así que falta que el contador
confirme en producción que la caída no se repite. Además, el enlace abre el listado de
comprobantes pero todavía no posiciona el comprobante específico.

---

### H2 · El Libro de Compras y Ventas no muestra los movimientos registrados como comprobante

| | |
|---|---|
| **Reportado** | Sesión S2, 22-10-2026 |
| **Severidad** | Media |
| **Estado** | En análisis |

**Descripción.** En el período octubre de 2026 el Libro de Compras responde "Sin
documentos de compra en este período", pese a que el Libro Diario sí muestra una
compra de mercaderías por 1.190.000 en ese mismo mes.

**Pasos para reproducir.** Registrar una compra desde Registro de Voucher, con línea
imputada a un proveedor, y luego generar el Libro de Compras del período.

**Análisis preliminar.** El libro lista documentos tributarios, no movimientos
contables. Un comprobante sin su documento asociado queda fuera. Falta confirmar si es
el comportamiento esperado o si debe leer también desde los vouchers con contraparte.

---

### H3 · Los montos se concatenan al corregirlos

| | |
|---|---|
| **Reportado** | Sesión S2, 22-10-2026 |
| **Severidad** | Media |
| **Estado** | Abierto |

**Descripción.** En el Registro de Voucher, al escribir sobre una casilla de Debe o
Haber que ya tiene un monto, el valor nuevo se concatena al anterior en vez de
reemplazarlo. Seleccionar el texto con triple clic no lo sustituye.

**Pasos para reproducir.** Escribir 250.000 en el Debe, luego hacer triple clic sobre
la casilla y escribir 250.000 otra vez: el campo queda en 1.250.003.250.000.

**Impacto.** Obliga a borrar con la tecla de retroceso. En un comprobante largo
aumenta el riesgo de dejar un monto equivocado sin notarlo.

---

### H4 · Las rutas inválidas redirigen al inicio sin aviso

| | |
|---|---|
| **Reportado** | Sesión S2, 22-10-2026 |
| **Severidad** | Baja |
| **Estado** | Abierto |

**Descripción.** Al entrar a una dirección que no corresponde a un módulo, el sistema
redirige al Home sin mostrar ningún mensaje. El usuario no sabe si se equivocó de
dirección o si el módulo no existe.

**Sugerencia.** Mostrar una pantalla de "página no encontrada" con un enlace al Home.

---

### H5 · El Formulario 22 muestra 0 de 6 cuentas de resultado mapeadas

| | |
|---|---|
| **Reportado** | Sesión S2, 22-10-2026 |
| **Severidad** | Alta |
| **Estado** | En análisis |

**Descripción.** El módulo Formulario 22 y DDJJ indica "0/6 cuentas de resultado
mapeadas", de modo que el documento pre-llenado no se puede generar todavía.

**Análisis preliminar.** Corresponde al mapeo de cuentas a los códigos oficiales, que
está planificado como spec propia y depende del catálogo que debe validar el contador.
Por decisión del equipo, este módulo se aborda al final, después del resto de las
specs en curso.

---

### H6 · *(plantilla para el próximo hallazgo)*

| | |
|---|---|
| **Reportado** | |
| **Severidad** | |
| **Estado** | Abierto |

**Descripción.**

**Pasos para reproducir.**

**Resolución.**

---

## 4. Acciones de habilitación

| Fecha | Acción | Estado |
|---|---|---|
| 09-10-2026 | Script de carga del cliente piloto y cambio de contraseña obligatorio en el primer ingreso (PR #103). | En `main` |
| *(por confirmar)* | Creación de la cuenta del contador en producción. | Según el roadmap se creó directamente en la base de datos; falta dejar constancia. |
| *(pendiente)* | Carga de credenciales del API Gateway en producción y conmutación a modo real. | Pendiente |

---

## 5. Observaciones del usuario que no son errores

Las sugerencias de mejora que surgen durante el uso se registran aquí y se evalúan
como requerimientos, no como defectos.

| # | Fecha | Observación | Decisión |
|---|---|---|---|
| O1 | 30-09-2026 | Al bajar el libro de compras, listar primero los documentos sin cuenta de gasto asignada. | Incorporado como requerimiento de la etapa siguiente. |
| O2 | 30-09-2026 | Mantener la última fecha registrada al ingresar comprobantes, en vez de volver al primer día del mes. | Incorporado como requerimiento. |
| O3 | 30-09-2026 | Crear y recuperar comprobantes tipo desde la misma pantalla, sin salir del módulo. | Por evaluar: la funcionalidad existe (SPEC-018), falta el ajuste de experiencia de uso. |

---

## 6. Estado general

- **Sesiones registradas:** 2 de 5 planificadas.
- **Hallazgos abiertos:** 5. H1 a la espera de confirmación del usuario; H2 y H5 en
  análisis; H3 y H4 pendientes de corrección.
- **Verificado en la sesión S2:** el ciclo contable funciona de punta a punta. El
  comprobante valida la cuadratura y bloquea el guardado cuando el Debe y el Haber no
  coinciden; el movimiento registrado aparece en el Libro Diario y en el Libro Mayor; y
  el Balance General cuadra sus ocho columnas, con el resultado del ejercicio
  coincidiendo con el Estado de Resultado.
- **Bloqueante para ampliar la marcha blanca:** la integración con el SII sigue en modo
  simulado en producción, de modo que las pruebas con documentos reales del contribuyente
  no se pueden ejecutar todavía.

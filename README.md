# Sigefa – Guía de facturación electrónica para salas de belleza

Este documento resume los conceptos esenciales para implementar facturación electrónica en Colombia (DIAN) dentro de un software de agenda y punto de venta orientado a salones de belleza.

## 1. Qué entender primero (DIAN)

1) **Generar el documento electrónico:** formato XML UBL 2.1 con anexos DIAN. Incluye emisor, receptor, IVA, base gravable, medio de pago, etc.
2) **Firmar digitalmente el XML:** certificado digital de la empresa emisora o del proveedor tecnológico.
3) **Enviar a la DIAN:** normalmente vía servicios web (SOAP). La DIAN responde con aceptado/rechazado/en proceso.
4) **Generar CUFE/CUDE:** código único calculado con datos de la factura (NIT, número, fecha, valor, etc.).
5) **Representación gráfica:** PDF con QR y CUFE para el cliente (impresión o envío por email/WhatsApp).

Puedes integrarte a un proveedor tecnológico autorizado o conectarte directamente a la DIAN (requiere habilitación y mantenimiento normativo).

## 2. Arquitectura pensada para una sala de belleza

- **Agenda/turnos:** citas, servicios (corte, tintura, manicure), colaboradores; debe permitir pasar una cita completada a venta/factura.
- **Punto de venta (POS):** selección rápida de servicios y productos, formas de pago (efectivo, tarjeta, Nequi, Bancolombia), descuentos y propinas.
- **Facturación electrónica:** toma la venta y genera factura, notas crédito/débito, firma, envía a DIAN y guarda estados.
- **Clientes:** datos básicos y historial de servicios.
- **Reportes:** ventas diarias/mensuales, por estilista, por servicio, facturas aceptadas/rechazadas.

## 3. Decisión clave: conexión a DIAN vs. proveedor tecnológico

- **Directo con DIAN:** control total y sin fee mensual, pero mayor complejidad (UBL, firma digital, habilitación, cambios normativos).
- **Proveedor tecnológico:** API más amigable (REST), gestión de normativas y firma; implica costo por factura/mensual y dependencia de tercero.

> Recomendación MVP: usar un proveedor tecnológico y evaluar conexión directa cuando el volumen crezca.

## 4. Flujo típico de facturación

1. Recepción crea cita con servicios y estilista.
2. El cliente se atiende y la venta se confirma en el POS.
3. El POS genera la venta con líneas de servicios/productos e impuestos aplicables.
4. El usuario elige **Generar factura electrónica**.
5. El sistema construye el XML requerido por DIAN/proveedor y llama la API.
6. Respuesta:
   - **Aceptado:** guardar CUFE, XML y PDF; ofrecer envío/impresión.
   - **Rechazado:** mostrar error (NIT inválido, numeración, etc.) y permitir reintentos.
7. Descargar/almacenar PDF y enviarlo al cliente (correo, WhatsApp o impresión).

## 5. Stack recomendado (ejemplo)

- **Backend:** Node.js (Express/NestJS) o Python (Django/FastAPI) para lógica, conexión DIAN/proveedor y seguridad.
- **Frontend:** React/Vue web; Electron o React Native/Flutter para POS táctil.
- **Base de datos:** PostgreSQL/MySQL; Redis opcional para caché/colas.
- **Infraestructura:** nube (AWS/GCP/Azure) o servidor local inicial; copias de seguridad y controles de seguridad.

## 6. Modelo de datos básico (simplificado)

- `empresas`, `usuarios`, `clientes`, `servicios`, `productos`, `empleados` (estilistas), `citas`.
- `ventas` (encabezado) y `ventas_detalle`.
- `facturas_electronicas`: `id_venta`, `cufe`, `xml_ruta`, `pdf_ruta`, `estado_dian` (aceptada/rechazada/en_proceso), `mensaje_dian`.

## 7. Errores frecuentes a prevenir

- Numeración de facturas sin control de rangos/resoluciones autorizadas.
- Tiempos de respuesta lentos en DIAN/proveedor: implementar reintentos y consultas de estado.
- Interfaces pesadas: la facturación debe requerir los mínimos datos para no frenar la operación del salón.

## 8. Plan de trabajo (MVP → producto)

1. **Fase 1 – MVP sin DIAN:** agenda, clientes, servicios, empleados, registro de ventas y reportes básicos.
2. **Fase 2 – Integración DIAN:** elegir proveedor o integración directa, configurar NIT/régimen/resoluciones, flujo de factura electrónica, almacenamiento de XML/PDF.
3. **Fase 3 – Extras salón:** comisiones por estilista, fichas de cliente (historial de color, tratamientos, alergias), recordatorios por WhatsApp/SMS y programas de fidelización.

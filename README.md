# AuthFlow

### Flujos de Aprobación Empresarial On-Chain | Powered by Soroban Authorization Trees

---

## El Problema

Cada día, millones de PYMEs en Latinoamérica mueven dinero de la peor forma posible.

Una compra de $5,000 USD en insumos para una empresa de 15 personas pasa por esto: el encargado de compras envía un WhatsApp al gerente. El gerente reenvía un email al CEO pidiendo aprobación. El CEO responde "ok" desde el celular. El tesorero abre el portal bancario, ingresa los datos del proveedor manualmente, y ejecuta la transferencia. Tiempo total: **2 a 5 días**. Costo oculto: horas-hombre, errores de transcripción, cero trazabilidad, y un riesgo permanente de fraude interno.

Los ERPs que resuelven esto  SAP, Oracle NetSuite, Coupa  cuestan entre **$500 y $5,000 USD/mes**. Son soluciones diseñadas para corporaciones Fortune 500, no para la ferretería de 12 empleados, la cooperativa agrícola de 30 socios, o la startup de 8 personas que mueve capital entre tres países.

El resultado: **el 78% de las PYMEs en LATAM gestiona sus aprobaciones financieras por canales informales**  WhatsApp, email, llamadas telefónicas. Sin auditoría, sin atomicidad, sin protección contra replay de autorizaciones.

Esto no es un problema de digitalización. Es un problema de **infraestructura financiera**.

---

## La Solución: AuthFlow

AuthFlow es una plataforma de flujos de aprobación financiera donde cada participante firma únicamente la parte que le corresponde, y el pago se ejecuta atómicamente solo cuando todas las autorizaciones requeridas están completas.

No es un ERP. No es un chatbot de aprobaciones. Es **infraestructura de autorización financiera programable** construida sobre las Authorization Trees de Soroban.

### Cómo funciona

1. **El administrador define el flujo de aprobación** como un árbol de autorización en un contrato Soroban:

```
compra > $1,000:
  ├── require_auth(encargado_compras)
  ├── require_auth(gerente_area)
  ├── require_auth(CEO)         // condicional: solo si > $5,000
  └── execute: transfer(USDC, proveedor, monto)
```

2. **Cada aprobador firma solo su parte.** El encargado de compras inicia la solicitud y firma SU nodo del árbol. El gerente recibe una notificación, revisa el monto y el proveedor, y firma su nodo. El CEO firma si el monto supera el umbral. Nadie necesita construir la transacción completa  solo autorizar lo que le corresponde. Esto es **Partial Authorization nativa de Soroban**.

3. **Ejecución atómica.** Cuando el último aprobador firma, el contrato ejecuta el pago al proveedor en USDC. Si falta una firma, los fondos no se mueven. Si alguien rechaza, nada pasa. Atomicidad total  no existe el estado intermedio de "aprobado pero no pagado".

4. **Replay Protection automática a nivel de host.** El encargado que aprobó la Orden #47 no puede reutilizar esa firma para la Orden #48. Soroban lo previene automáticamente a nivel de protocolo, sin código adicional. Zero riesgo de doble autorización.

### Extensiones del producto

- **Aprobaciones condicionales por monto:** umbrales configurables (< $1K: 1 firma; $1K-$5K: 2 firmas; > $5K: 3 firmas + CEO).
- **Firma con Passkeys:** cada aprobador firma con FaceID o huella digital. Sin seed phrases, sin wallets complejas.
- **Fee-Bump patrocinado:** la empresa paga todos los fees. Los aprobadores nunca necesitan XLM.
- **Audit trail inmutable:** cada autorización queda registrada on-chain con timestamp, firmante, y contexto de la operación.
- **Variante DeFi:** vaults tipo DeFindex donde 2-de-3 fund managers deben autorizar un rebalanceo de portafolio. El quórum se alcanza con partial auth, y el rebalance se ejecuta atómicamente.

---

## Público Objetivo

### Segmento primario: PYMEs LATAM (5–50 empleados)

- Empresas con 3+ personas involucradas en decisiones de gasto.
- Operan con proveedores internacionales o regionales (pagos cross-border).
- No pueden justificar un ERP de $500+/mes pero necesitan control financiero.
- Sectores: comercio, manufactura, agro, servicios profesionales, cooperativas.

### Segmento secundario: DAOs y tesorerías crypto-nativas

- Organizaciones descentralizadas que gestionan fondos colectivos.
- Necesitan aprobación multi-parte para movimientos de tesorería.
- Hoy usan Gnosis Safe ($10-30 por transacción) o procesos manuales.

### Segmento terciario: Fintechs y neobancos como infraestructura

- AuthFlow como API/SDK que fintechs integran en sus productos.
- White-label: "Aprobaciones empresariales" como feature dentro de una plataforma financiera existente.

### Tamaño de mercado

- **LATAM tiene 30+ millones de PYMEs.** El 90% no usa software de gestión de aprobaciones.
- El mercado global de Business Process Management (BPM) financiero es de **$16B USD** y crece 12% anual.
- AuthFlow ataca el segmento desatendido: empresas demasiado pequeñas para SAP, demasiado grandes para WhatsApp.

---

## Por qué Stellar es la Blockchain Adecuada

AuthFlow no "usa blockchain porque sí". Explota **cuatro primitivas exclusivas de Soroban** que hacen que esta solución sea técnicamente imposible o económicamente inviable en cualquier otra cadena.

### 1. Authorization Trees con Partial Auth  EXCLUSIVO de Soroban

Soroban tiene un sistema de autorización a nivel de host donde cada participante firma únicamente su rama del árbol de autorización. No necesitan ver ni construir la transacción completa.

**En Ethereum:** el patrón es `approve()` → `transferFrom()`. Dos transacciones separadas, vulnerables a frontrunning, sin atomicidad entre la aprobación y la ejecución. Para replicar partial auth necesitas un contrato custom tipo Gnosis Safe ($50-200 de deploy, $10-30 por transacción de aprobación).

**En Soroban:** `require_auth()` en cada nodo del árbol. Una sola transacción atómica. **$0.001 total.**

### 2. Replay Protection Automática  Built-in a Nivel de Protocolo

Soroban previene automáticamente que la misma autorización se reutilice. Es una garantía del runtime, no código que el desarrollador escribe.

**En Ethereum:** implementas nonces manuales en cada contrato. Si te equivocas, tienes un bug de seguridad crítico. En Soroban: es gratis y automático.

### 3. Sin approve/transferFrom  El Anti-Pattern de EVM Eliminado

El modelo de auth de Soroban mueve fondos directamente bajo autorización parcial, sin el paso intermedio de "aprobar tokens para que otro contrato los gaste". Esto elimina una superficie de ataque completa (infinite approval exploits) que en EVM ha causado **$2B+ en hacks**.

### 4. Transacciones Atómicas Multi-Operación

El pago al proveedor + el registro contable + la actualización del presupuesto del departamento ocurren en **una sola transacción**. Si cualquier parte falla, todo se revierte. En Ethereum, cada operación es una transacción separada con su propio fee y su propio riesgo de falla parcial.

### Comparativa de costos

| Operación | AuthFlow (Stellar) | Gnosis Safe (Ethereum) | ERP Tradicional |
|---|---|---|---|
| Crear flujo de aprobación | $0.01 (deploy contrato) | $50-200 (deploy Safe) | $500-5,000/mes |
| Cada aprobación | $0.001 | $10-30 | Incluido en suscripción |
| 100 aprobaciones/mes | $0.10 | $1,000-3,000 | $500-5,000/mes |
| Replay protection | Gratis (protocolo) | Manual (código custom) | N/A |
| Latencia por aprobación | 5 segundos | 15-60 segundos | Variable (email) |
| Audit trail | On-chain inmutable | On-chain inmutable | Base de datos privada |

---

## Impacto Potencial

### Impacto económico inmediato

- **Reducción de 80% en tiempo de aprobación** de gastos (de 2-5 días a minutos).
- **Eliminación de errores de transcripción** en pagos a proveedores (el pago es directo desde el contrato, no requiere ingreso manual de datos bancarios).
- **Ahorro de $500-5,000/mes** vs. ERPs tradicionales para PYMEs que hoy no pueden acceder a estas soluciones.
- **$0.10/mes** en costos operativos para 100 aprobaciones mensuales vs. $1,000-3,000 en Ethereum.

### Impacto en transparencia y gobernanza

- **Audit trail inmutable y público:** cada aprobación queda registrada on-chain. Elimina la posibilidad de alterar registros contables retroactivamente.
- **Prevención de fraude interno:** la replay protection automática y la atomicidad hacen imposible que un empleado reutilice una autorización o ejecute un pago no aprobado.
- **Compliance simplificado:** los auditores pueden verificar la cadena de aprobaciones directamente en el ledger, sin depender de reportes generados por la empresa.

### Impacto en inclusión financiera

- AuthFlow permite que **cualquier PYME**, sin importar su tamaño o presupuesto tecnológico, tenga el mismo nivel de control financiero que una corporación con SAP.
- La firma con Passkeys (biometría) elimina la barrera de adopción crypto: el CFO firma con FaceID, no con una seed phrase de 24 palabras.
- Los Sponsored Reserves y Fee-Bumps significan que **nadie en la empresa necesita comprar, tener o entender XLM**.

### Visión a 3 años

AuthFlow empieza como una herramienta de aprobación de gastos. Pero la infraestructura de Authorization Trees es genérica  cualquier flujo de autorización multi-parte puede construirse sobre ella:

- **Aprobaciones de crédito** en cooperativas de ahorro.
- **Firmas multi-parte** para contratos legales tokenizados.
- **Governance de tesorerías** para DAOs, ONGs, y gobiernos locales.
- **Supply chain finance:** órdenes de compra que fluyen por la cadena de suministro con aprobación en cada eslabón.

El objetivo es que AuthFlow se convierta en el **estándar de autorización financiera multi-parte en Stellar**  la capa que cualquier aplicación integra cuando necesita "más de una persona debe aprobar esto".

---

## Modelo de Negocio

| Canal | Modelo | Precio estimado |
|---|---|---|
| **SaaS directo** | Suscripción mensual por empresa | $29-99/mes según # de usuarios |
| **API/SDK** | Por transacción de aprobación procesada | $0.05-0.10 por aprobación |
| **White-label** | Licencia para fintechs y neobancos | $500-2,000/mes por integración |
| **Enterprise** | Implementación custom + soporte | Desde $5,000 setup + mensual |

El margen operativo es alto porque el costo marginal de cada aprobación en Stellar es $0.001. Incluso cobrando $0.05 por aprobación, el margen es de **98%**.

---

## Stack Técnico

- **Smart Contracts:** Soroban (Rust)  contratos de flujo de aprobación con Authorization Trees
- **Autenticación:** Passkeys (Secp256r1) nativos de Stellar  firma biométrica
- **Fees:** Fee-Bump Transactions  la empresa paga todo, los empleados nunca ven fees
- **Almacenamiento:** Persistent Storage de Soroban para flujos activos; State Archival automática para flujos completados
- **Frontend:** App móvil (React Native) + Dashboard web (Next.js)
- **Notificaciones:** Push notifications + integración Slack/Teams para solicitudes pendientes
- **Pagos:** USDC en Stellar como moneda de ejecución; Path Payments para conversión automática si el proveedor opera en otra moneda

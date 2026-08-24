# Investigación: Aplicaciones de Blockchain y Smart Contracts

---

## 1. Oportunidad de dApp para Perú: Plataforma de Trazabilidad Agroexportadora (*AgroTrace Perú*)

### 1.1 El Problema que Resuelve
Perú es uno de los principales exportadores mundiales de productos agrícolas de alto valor como el **café de especialidad, cacao fino de aroma, uvas de mesa, paltas Hass y espárragos**. Sin embargo, la cadena de valor enfrenta graves desafíos:

- **Falsificación de Certificaciones de Origen y Orgánicas:** La obtención de sellos como *Organic*, *Fair Trade* o *Rainforest Alliance* depende de entidades certificadoras centralizadas cuyo proceso es costoso, lento y vulnerable a la falsificación de documentos físicos en papel o PDFs alterables.
- **Falta de Transparencia para Compradores Internacionales:** Los importadores en Europa, Estados Unidos y Asia exigen garantías verificables sobre el origen del lote, el no uso de tierras deforestadas (cumplimiento del reglamento UE 2023/1115 de la Unión Europea) y las condiciones de almacenamiento (cadena de frío).
- **Injusta Distribución de Ingresos:** Los pequeños agricultores en regiones como Junín, San Martín o Cusco reciben una fracción reducida del precio final debido a la excesiva intermediación y la imposibilidad de demostrar individualmente la calidad superior de su cosecha.

---

### 1.2 Por qué Blockchain es mejor que un Sistema Tradicional
Los sistemas tradicionales de gestión de cadena de suministro (ERP centralizados o bases de datos SQL relacionales) presentan limitaciones estructurales frente a una arquitectura Blockchain:

| Criterio | Sistema Tradicional (ERP / SQL) | Plataforma Blockchain (dApp) |
| :--- | :--- | :--- |
| **Control de Datos** | Administrado por una sola entidad (exportadora o certificadora) que puede alterar registros retroactivamente. | Registro distribuido e inmutable donde ningún participante puede modificar el historial registrado. |
| **Confianza y Auditoría** | Requiere auditorías presenciales y revisión manual de documentos físicos. | Criptográficamente verificable en tiempo real por cualquier comprador escaneando un código QR. |
| **Interoperabilidad** | Silos de información incomunicados entre agricultores, transportistas, Senasa y aduanas. | Registro único compartido con estándares abiertos y acceso descentralizado. |
| **Automatización Financiera** | Pagos bancarios manuales a 60-90 días tras la entrega. | Pagos automáticos condicionantes (*Escrow* mediante Smart Contracts) ejecutados al validar la llegada del lote. |

---

### 1.3 Funcionamiento a Alto Nivel de la dApp (*AgroTrace Perú*)

```
[1. Cosecha] ➔ [2. Acopio & Senasa] ➔ [3. Procesamiento & Embarque] ➔ [4. Destino Final]
  (Agricultor)     (Firma Digital IoT)      (Exportador / Aduanas)       (Comprador global)
       │                    │                         │                          │
       └────────────────────┴──────────┬──────────────┴──────────────────────────┘
                                       ▼
                       Smart Contract AgroTrace (Blockchain)
                                       │
                                       ▼
                   Tokenización NFT / Pasaporte Digital del Lote
```

1. **Registro e Identificación del Lote (Cosecha):**
   El agricultor registra la cosecha desde su teléfono móvil. Se genera un **Pasaporte Digital de Producto (NFT ERC-721 o ERC-1155)** que incluye geolocalización de la parcela, fecha de recolección y volumen.
2. **Control Fitosanitario y Acopio (Procesamiento):**
   Al ingresar al centro de acopio o planta procesadora, inspectores y sensores IoT (temperatura/humedad) envían datos que quedan registrados inmutablemente en la blockchain mediante Smart Contracts. Inspecciones de SENASA firman criptográficamente la conformidad del lote.
3. **Exportación y Logística Marítima:**
   En el puerto (ej. Puerto de Callao o Chancay), la aglomeración de lotes en contenedores se asocia a contratos inteligentes de flete, registrando el manifiesto de aduanas y el conocimiento de embarque (*Bill of Lading*).
4. **Verificación y Pago Inmediato en Destino:**
   El comprador en el puerto de destino (ej. Rotterdam) escanea el código QR del contenedor/saco. El Smart Contract verifica que la cadena de frío no se rompió y que el origen es legítimo, liberando automáticamente los fondos en *Stablecoins* (USDC/USDT) directamente al agricultor y exportador.

---

## 2. Casos de Uso Reales y Actuales de Smart Contracts en dApps

---

### 2.1 Caso 1: Uniswap (Automated Market Maker - AMM)

#### A. Problema que Resuelve
En las finanzas tradicionales y en los intercambios centralizados de criptomonedas (CEX como Binance o Coinbase), los intercambios de activos dependen de un **Libro de Órdenes (Order Book)** y de creadores de mercado (*Market Makers*) institucionales. Esto exige confiar la custodia de las fondos a un tercero intermediario, con riesgos de censura, congelamiento de fondos y falta de transparencia en la liquidez.

#### B. Cómo Funciona el Smart Contract
Uniswap elimina el libro de órdenes introduciendo el modelo de **Creador de Mercado Automatizado (AMM)** gestionado íntegramente por Smart Contracts inmutables desplegados en Ethereum y redes EVM:

1. **Pools de Liquidez (Liquidity Pools):** Los contratos inteligentes guardan reservas de dos tokens (por ejemplo, ETH/USDC). Cualquier usuario puede convertirse en Proveedor de Liquidez (LP) depositando ambos tokens en proporción igualitaria a cambio de tokens de liquidez (LP Tokens).
2. **Fórmula del Producto Constante ($x \cdot y = k$):** El precio de swap se determina matemáticamente por la fórmula:
   $$x \cdot y = k$$
   Donde $x$ es la cantidad del Token A, $y$ es la cantidad del Token B, y $k$ es una constante fija. Cuando un usuario compra Token A introduciendo Token B en el contrato, la cantidad de Token A disminuye y la de Token B aumenta, ajustando automáticamente el precio según la oferta y la demanda.
3. **Ejecución Atómica:** La transacción de intercambio (swap) se ejecuta en un solo bloque. Si las condiciones de deslizamiento de precio (*slippage*) se superan, la transacción completa se revierte automáticamente, protegiendo los fondos del usuario.

#### C. Por qué es un Ejemplo Relevante
Uniswap es la dApp de finanzas descentralizadas (DeFi) más influyente de la historia. Ha procesado más de **2 billones de dólares en volumen acumulado** sin haber sufrido hackeos en sus contratos del núcleo (*core*), demostrando que es posible operar un mercado financiero global de liquidez abierta las 24 horas del día, los 7 días de la semana, de forma no custodial y 100% automatizada.

---

### 2.2 Caso 2: VeChain y IBM Food Trust (Trazabilidad de Alimentos en Cadena de Suministro)

#### A. Problema que Resuelve
Las cadenas de suministro globales de alimentos son opacas y fragmentadas. Ante brotes de bacterias (como *E. coli* o *Salmonella* en lechugas o carnes), las autoridades sanitarias y supermercados tardan **semanas** en rastrear el origen exacto del lote contaminado. Esto provoca el desperdicio masivo de toneladas de alimento seguro retirado de forma indiscriminate y pérdidas millonarias para los productores.

#### B. Cómo Funciona el Smart Contract
Redes como **VeChainThor** y **IBM Food Trust (Hyperledger Fabric)** utilizan contratos inteligentes para orquestar la trazabilidad:

1. **Enlace Criptográfico Físico-Digital (IoT + Blockchain):** Los lotes de alimentos se etiquetan con chips RFID, sensores IoT o códigos QR criptográficos. Cada vez que el producto pasa de la granja al camión de refrigeración, al centro de distribución y finalmente al anaquel del supermercado, los dispositivos IoT envían lecturas de temperatura y ubicación firmadas digitalmente hacia el Smart Contract.
2. **Verificación de Condiciones Contratuales:** El Smart Contract evalúa automáticamente si las condiciones requeridas se cumplieron (por ejemplo: *"La temperatura del contenedor de carne nunca debe superar los 4 °C"*). Si el sensor registra una desviación térmica de 8 °C durante más de 30 minutos, el Smart Contract invalida automáticamente el sello de calidad del lote y alerta al distribuidor antes de que llegue al consumidor.
3. **Consultas de Auditoría:** Toda la historia queda registrada mediante hashes criptográficos indexados en la cadena de bloques.

#### C. Por qué es un Ejemplo Relevante
Este modelo está implementado en producción real por gigantes de la industria como **Walmart, Carrefour, Nestlé, Tyson Foods y Bayer**. En pruebas documentadas por Walmart e IBM Food Trust, el tiempo necesario para rastrear el origen de un paquete de mangos desde el anaquel del supermercado hasta la granja de origen se redujo de **7 días a solo 2.2 segundos**, demostrando la eficiencia de los Smart Contracts en la seguridad alimentaria mundial.

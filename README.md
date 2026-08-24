# AgroTrace Perú - Conector Pali Wallet (dApp Svelte + Ethers v6)

Este repositorio contiene la entrega completa para el trabajo de dApps y Smart Contracts, compuesto por:
1. **Investigación Teórica (`INVESTIGACION.md`):** Análisis detallado de una propuesta de dApp de trazabilidad agroexportadora para Perú y el funcionamiento técnico de Smart Contracts en casos de uso reales (**Uniswap** y **VeChain / IBM Food Trust**).
2. **Demo Práctica Web3 (Svelte + Vite):** Aplicación funcional que se conecta a la billetera **Pali Wallet** usando la librería **Ethers.js v6** bajo el estándar **EIP-1193** (`window.ethereum`).

---

## 🚀 Características de la Demo

- 🔐 **Conexión con Pali Wallet:** Detecta la inyección de `window.ethereum` y solicita permisos mediante `eth_requestAccounts`.
- 📬 **Lectura de Dirección (Address):** Obtiene la cuenta activa de la billetera utilizando `provider.getSigner()` y `signer.getAddress()`.
- 💰 **Consulta de Saldo (Balance):** Consulta el saldo nativo mediante `provider.getBalance(address)` y lo da formato en Ether con `ethers.formatEther()`.
- 🚨 **Manejo de Errores y Estados Reactivos:**
  - Alerta interactiva cuando Pali Wallet no está instalada en el navegador.
  - Captura y manejo accesible cuando el usuario rechaza la solicitud de firma/conexión.
  - Actualización automática de datos ante eventos de cambio de cuenta o red (`accountsChanged`, `chainChanged`).
  - Copiado rápido de la dirección al portapapeles.

---

## 📋 Requisitos Previos

Antes de ejecutar el proyecto, asegúrate de contar con:

1. **Node.js** (versión 18 o superior recomendada) y **npm**.
2. **Navegador Web** (Chrome, Brave, Edge o Firefox).
3. **Extensión Pali Wallet** instalada en tu navegador:
   - Puedes descargar la extensión desde el sitio oficial: [https://paliwallet.com/](https://paliwallet.com/)
   - Crea o importa una billetera y conéctala a la red de tu preferencia (ej. Syscoin EVM, Ethereum Mainnet o Sepolia Testnet).

---

## 🔧 Instalación y Ejecución Local

Sigue estos pasos para clonar e iniciar el proyecto en tu máquina local:

```bash
# 1. Navegar al directorio del proyecto
cd pali-svelte-dapp

# 2. Instalar dependencias
npm install

# 3. Iniciar el servidor de desarrollo
npm run dev
```

El servidor local se iniciará normalmente en `http://localhost:5173`. Abre este enlace en el navegador donde instalaste la extensión **Pali Wallet**.

### Otros Comandos Útiles

- **Compilar para producción:**
  ```bash
  npm run build
  ```
- **Previsualizar compilación de producción:**
  ```bash
  npm run preview
  ```

---

## 📁 Estructura del Proyecto

```
pali-svelte-dapp/
├── INVESTIGACION.md      # Documentación teórica de dApp AgroTrace Perú y casos de uso
├── README.md             # Instrucciones de uso e instalación
├── package.json          # Gestión de dependencias y scripts del proyecto
├── vite.config.js        # Configuración del bundler Vite para Svelte
├── index.html            # Plantilla HTML principal
└── src/
    ├── main.js           # Punto de entrada JavaScript
    ├── app.css           # Estilos Dark Glassmorphic Web3
    └── App.svelte        # Componente reactivo con Ethers.js v6 y Pali Wallet
```

---

## 📄 Archivo de Investigación

Para revisar la sección teórica completa (propuesta de dApp para Perú y análisis de Smart Contracts en Uniswap y VeChain / IBM Food Trust), abre el archivo [`INVESTIGACION.md`](INVESTIGACION.md).

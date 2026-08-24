<script>
  import { onMount } from 'svelte';
  import { ethers } from 'ethers';

  // Reactive State
  let isInstalled = false;
  let isConnecting = false;
  let isConnected = false;
  let address = '';
  let balance = '';
  let networkName = '';
  let chainId = null;
  let errorMessage = '';
  let copySuccess = false;

  // Formateador de dirección corta (0x1234...abcd)
  function formatAddress(addr) {
    if (!addr) return '';
    return `${addr.slice(0, 6)}...${addr.slice(-4)}`;
  }

  // Comprobar la inyección de window.ethereum (Pali Wallet / EIP-1193)
  function checkWalletInstalled() {
    if (typeof window !== 'undefined' && window.ethereum) {
      isInstalled = true;
      return true;
    } else {
      isInstalled = false;
      return false;
    }
  }

  onMount(async () => {
    const installed = checkWalletInstalled();

    if (installed) {
      // Escuchar eventos de cambio de cuenta o red en la billetera
      if (window.ethereum.on) {
        window.ethereum.on('accountsChanged', handleAccountsChanged);
        window.ethereum.on('chainChanged', () => window.location.reload());
      }

      // Verificación automática silenciosa de cuentas autorizadas previamente
      try {
        const provider = new ethers.BrowserProvider(window.ethereum);
        const accounts = await provider.send('eth_accounts', []);
        if (accounts && accounts.length > 0) {
          await loadWalletData(provider, accounts[0]);
        }
      } catch (err) {
        console.warn('Verificación inicial de autorización:', err);
      }
    }
  });

  async function handleAccountsChanged(accounts) {
    if (!accounts || accounts.length === 0) {
      disconnectWallet();
    } else {
      const provider = new ethers.BrowserProvider(window.ethereum);
      await loadWalletData(provider, accounts[0]);
    }
  }

  async function loadWalletData(provider, userAddress = null) {
    try {
      errorMessage = '';
      const signer = await provider.getSigner();
      const currentAddress = userAddress || (await signer.getAddress());

      // Ethers.js v6 balance query & formatting
      const balanceWei = await provider.getBalance(currentAddress);
      const network = await provider.getNetwork();

      address = currentAddress;
      balance = parseFloat(ethers.formatEther(balanceWei)).toFixed(4);
      networkName = network.name !== 'unknown' ? network.name : `Chain ID ${network.chainId}`;
      chainId = Number(network.chainId);
      isConnected = true;
    } catch (err) {
      console.error('Error al cargar datos de la billetera:', err);
      errorMessage = 'No se pudo obtener la dirección o el saldo de la billetera.';
    }
  }

  async function connectWallet() {
    errorMessage = '';

    if (!checkWalletInstalled()) {
      errorMessage = 'Pali Wallet no está instalada en tu navegador. Por favor instala la extensión.';
      return;
    }

    isConnecting = true;

    try {
      // Crear Provider Ethers v6 sobre window.ethereum (EIP-1193)
      const provider = new ethers.BrowserProvider(window.ethereum);
      
      // Solicitar acceso a cuentas
      const accounts = await provider.send('eth_requestAccounts', []);

      if (accounts && accounts.length > 0) {
        await loadWalletData(provider, accounts[0]);
      } else {
        errorMessage = 'No se seleccionaron cuentas en la solicitud de conexión.';
      }
    } catch (err) {
      console.error('Error durante la conexión:', err);
      if (err.code === 4001 || (err.message && err.message.toLowerCase().includes('reject'))) {
        errorMessage = 'Solicitud de conexión rechazada por el usuario en Pali Wallet.';
      } else {
        errorMessage = err.message || 'Ocurrió un error inesperado al conectar con la billetera.';
      }
    } finally {
      isConnecting = false;
    }
  }

  function disconnectWallet() {
    isConnected = false;
    address = '';
    balance = '';
    networkName = '';
    chainId = null;
    errorMessage = '';
  }

  async function copyToClipboard() {
    if (!address) return;
    await navigator.clipboard.writeText(address);
    copySuccess = true;
    setTimeout(() => {
      copySuccess = false;
    }, 2000);
  }

  async function refreshBalance() {
    if (!isConnected || !window.ethereum) return;
    const provider = new ethers.BrowserProvider(window.ethereum);
    await loadWalletData(provider, address);
  }
</script>

<main class="container">
  <!-- Header / Badge -->
  <header class="header">
    <div class="badge">
      <span class="dot"></span>
      <span>AgroTrace Perú dApp</span>
    </div>
    <h1>Conector de Billetera Web3</h1>
    <p class="subtitle">Conexión con Pali Wallet mediante Ethers.js v6 y EIP-1193</p>
  </header>

  <!-- Notification Banner for Missing Wallet -->
  {#if !isInstalled}
    <div class="card alert-box warning">
      <div class="alert-icon">⚠️</div>
      <div class="alert-content">
        <h3>Pali Wallet no detectada</h3>
        <p>No se encontró <code>window.ethereum</code> en tu navegador. Necesitas la extensión <strong>Pali Wallet</strong> para interactuar con esta dApp.</p>
        <a 
          href="https://paliwallet.com/" 
          target="_blank" 
          rel="noopener noreferrer" 
          class="btn-link"
        >
          Descargar Pali Wallet &rarr;
        </a>
      </div>
    </div>
  {/if}

  <!-- Error Message Alert -->
  {#if errorMessage}
    <div class="card alert-box error">
      <div class="alert-icon">🚨</div>
      <div class="alert-content">
        <h3>Error de Conexión</h3>
        <p>{errorMessage}</p>
      </div>
    </div>
  {/if}

  <!-- Main Card State -->
  <div class="card main-card">
    {#if !isConnected}
      <!-- State 1: Disconnected -->
      <div class="state-disconnected">
        <div class="wallet-icon-wrapper">
          <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
            <path d="M3 9h18"/>
            <path d="M15 15h2"/>
          </svg>
        </div>
        <h2>Conecta tu Billetera</h2>
        <p class="description">
          Conecta Pali Wallet para autenticarte y consultar la dirección activa y tu saldo de forma segura en la blockchain.
        </p>

        <button 
          on:click={connectWallet} 
          disabled={isConnecting}
          class="btn-primary"
        >
          {#if isConnecting}
            <span class="spinner"></span>
            Conectando...
          {:else}
            🔐 Conectar Pali Wallet
          {/if}
        </button>
      </div>
    {:else}
      <!-- State 2: Connected -->
      <div class="state-connected">
        <div class="status-header">
          <div class="status-indicator">
            <span class="pulse-dot"></span>
            <span class="status-text">Conectado a {networkName}</span>
          </div>
          <button on:click={disconnectWallet} class="btn-disconnect" title="Desconectar">
            Desconectar
          </button>
        </div>

        <!-- Balance display section -->
        <div class="balance-display">
          <span class="balance-label">Saldo Disponible</span>
          <div class="balance-value">
            <span class="amount">{balance}</span>
            <span class="currency">ETH / SYS</span>
          </div>
          <button on:click={refreshBalance} class="btn-refresh" title="Actualizar saldo">
            🔄 Actualizar
          </button>
        </div>

        <!-- Address Box Section -->
        <div class="info-group">
          <span class="info-label">Dirección Conectada (Address):</span>
          <div class="address-box">
            <span class="mono-address" title={address}>
              {formatAddress(address)}
            </span>
            <button on:click={copyToClipboard} class="btn-copy">
              {#if copySuccess}
                ✓ Copiado
              {:else}
                📋 Copiar
              {/if}
            </button>
          </div>
          <p class="full-address-sub">{address}</p>
        </div>

        <!-- Chain Info Grid -->
        <div class="grid-details">
          <div class="detail-item">
            <span class="detail-label">Red</span>
            <span class="detail-val">{networkName}</span>
          </div>
          <div class="detail-item">
            <span class="detail-label">Chain ID</span>
            <span class="detail-val">#{chainId}</span>
          </div>
          <div class="detail-item">
            <span class="detail-label">Protocolo</span>
            <span class="detail-val">EIP-1193 (Ethers v6)</span>
          </div>
        </div>
      </div>
    {/if}
  </div>

  <!-- Footer -->
  <footer class="footer">
    <p>AgroTrace Perú dApp &copy; 2026 - Conector Pali Wallet &bull; Ethers.js v6</p>
  </footer>
</main>

<style>
  .container {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .header {
    text-align: center;
  }

  .badge {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    background: rgba(16, 185, 129, 0.1);
    border: 1px solid rgba(16, 185, 129, 0.3);
    color: var(--primary-green);
    font-size: 0.8rem;
    font-weight: 600;
    padding: 6px 14px;
    border-radius: 999px;
    margin-bottom: 12px;
  }

  .dot {
    width: 8px;
    height: 8px;
    background-color: var(--primary-green);
    border-radius: 50%;
    box-shadow: 0 0 8px var(--primary-green);
  }

  h1 {
    font-size: 1.85rem;
    font-weight: 800;
    color: #ffffff;
    letter-spacing: -0.02em;
    margin-bottom: 6px;
  }

  .subtitle {
    color: var(--text-muted);
    font-size: 0.95rem;
  }

  .card {
    background: var(--card-bg);
    border: 1px solid var(--card-border);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border-radius: 20px;
    padding: 28px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  }

  .alert-box {
    display: flex;
    gap: 16px;
    align-items: flex-start;
    padding: 18px 22px;
  }

  .alert-box.warning {
    background: var(--warning-bg);
    border-color: var(--warning-border);
    color: var(--warning-text);
  }

  .alert-box.error {
    background: var(--error-bg);
    border-color: var(--error-border);
    color: var(--error-text);
  }

  .alert-icon {
    font-size: 1.5rem;
    line-height: 1;
  }

  .alert-content h3 {
    font-size: 1rem;
    font-weight: 700;
    margin-bottom: 4px;
  }

  .alert-content p {
    font-size: 0.88rem;
    line-height: 1.4;
    margin-bottom: 8px;
  }

  .btn-link {
    display: inline-block;
    color: var(--primary-green);
    font-weight: 600;
    font-size: 0.88rem;
    text-decoration: none;
  }

  .btn-link:hover {
    text-decoration: underline;
  }

  .state-disconnected {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    padding: 10px 0;
  }

  .wallet-icon-wrapper {
    width: 72px;
    height: 72px;
    border-radius: 20px;
    background: rgba(16, 185, 129, 0.1);
    border: 1px solid rgba(16, 185, 129, 0.2);
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--primary-green);
    margin-bottom: 18px;
  }

  .state-disconnected h2 {
    font-size: 1.35rem;
    font-weight: 700;
    margin-bottom: 8px;
  }

  .description {
    color: var(--text-muted);
    font-size: 0.9rem;
    line-height: 1.5;
    max-width: 400px;
    margin-bottom: 24px;
  }

  .btn-primary {
    width: 100%;
    padding: 14px 24px;
    background: linear-gradient(135deg, var(--primary-green), var(--primary-green-hover));
    color: #ffffff;
    font-family: inherit;
    font-size: 1rem;
    font-weight: 700;
    border: none;
    border-radius: 12px;
    cursor: pointer;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    box-shadow: 0 4px 14px rgba(16, 185, 129, 0.35);
  }

  .btn-primary:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(16, 185, 129, 0.45);
  }

  .btn-primary:disabled {
    opacity: 0.7;
    cursor: not-allowed;
  }

  .spinner {
    width: 18px;
    height: 18px;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-top-color: #ffffff;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  /* Connected State Styles */
  .status-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }

  .status-indicator {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.85rem;
    font-weight: 600;
    color: var(--primary-green);
  }

  .pulse-dot {
    width: 10px;
    height: 10px;
    background-color: var(--primary-green);
    border-radius: 50%;
    position: relative;
  }

  .pulse-dot::after {
    content: '';
    position: absolute;
    top: -2px;
    left: -2px;
    right: -2px;
    bottom: -2px;
    border-radius: 50%;
    border: 2px solid var(--primary-green);
    animation: pulse 2s infinite;
  }

  @keyframes pulse {
    0% { transform: scale(0.95); opacity: 0.8; }
    70% { transform: scale(1.5); opacity: 0; }
    100% { transform: scale(0.95); opacity: 0; }
  }

  .btn-disconnect {
    background: transparent;
    border: 1px solid rgba(255, 255, 255, 0.15);
    color: var(--text-muted);
    font-family: inherit;
    font-size: 0.8rem;
    padding: 6px 12px;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .btn-disconnect:hover {
    background: rgba(239, 68, 68, 0.15);
    border-color: rgba(239, 68, 68, 0.4);
    color: #fca5a5;
  }

  .balance-display {
    background: rgba(0, 0, 0, 0.25);
    border: 1px solid rgba(255, 255, 255, 0.05);
    border-radius: 16px;
    padding: 20px;
    text-align: center;
    margin-bottom: 20px;
    position: relative;
  }

  .balance-label {
    font-size: 0.8rem;
    color: var(--text-muted);
    text-transform: uppercase;
    letter-spacing: 0.05em;
    font-weight: 600;
  }

  .balance-value {
    margin: 8px 0 12px 0;
  }

  .amount {
    font-size: 2.2rem;
    font-weight: 800;
    color: #ffffff;
    font-family: var(--font-mono);
  }

  .currency {
    font-size: 1rem;
    font-weight: 700;
    color: var(--accent-cyan);
    margin-left: 6px;
  }

  .btn-refresh {
    background: rgba(255, 255, 255, 0.06);
    border: none;
    color: var(--text-muted);
    font-size: 0.78rem;
    font-family: inherit;
    padding: 4px 10px;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.2s ease;
  }

  .btn-refresh:hover {
    background: rgba(255, 255, 255, 0.12);
    color: var(--text-main);
  }

  .info-group {
    margin-bottom: 20px;
  }

  .info-label {
    display: block;
    font-size: 0.82rem;
    color: var(--text-muted);
    font-weight: 600;
    margin-bottom: 6px;
  }

  .address-box {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 10px;
    padding: 10px 14px;
  }

  .mono-address {
    font-family: var(--font-mono);
    font-weight: 600;
    color: var(--accent-cyan);
    font-size: 0.95rem;
  }

  .btn-copy {
    background: rgba(255, 255, 255, 0.08);
    border: none;
    color: var(--text-main);
    font-family: inherit;
    font-size: 0.78rem;
    padding: 4px 10px;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.2s ease;
  }

  .btn-copy:hover {
    background: rgba(255, 255, 255, 0.16);
  }

  .full-address-sub {
    font-family: var(--font-mono);
    font-size: 0.72rem;
    color: var(--text-muted);
    margin-top: 6px;
    word-break: break-all;
    opacity: 0.7;
  }

  .grid-details {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    background: rgba(0, 0, 0, 0.15);
    border-radius: 12px;
    padding: 12px;
  }

  .detail-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .detail-label {
    font-size: 0.72rem;
    color: var(--text-muted);
    text-transform: uppercase;
    font-weight: 600;
  }

  .detail-val {
    font-size: 0.82rem;
    font-weight: 700;
    color: var(--text-main);
    margin-top: 2px;
  }

  .footer {
    text-align: center;
    color: var(--text-muted);
    font-size: 0.78rem;
    opacity: 0.7;
  }
</style>

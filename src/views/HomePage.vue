<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Crypto Prices</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="fetchData" fill="clear">
            <ion-icon slot="icon-only" name="refresh" />
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <!-- Search/Filter -->
      <ion-searchbar
        v-model="searchQuery"
        placeholder="Cari crypto (nama/symbol)..."
        debounce="300"
        class="mb-4"
      />

      <!-- Loading Skeleton -->
      <div v-if="loading && !error" class="skeleton-list">
        <div v-for="i in 5" :key="i" class="skeleton-row">
          <div class="skeleton-cell rank" />
          <div class="skeleton-cell name" />
          <div class="skeleton-cell symbol" />
          <div class="skeleton-cell price" />
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="error-state">
        <ion-icon name="alert-circle-outline" size="large" color="danger" />
        <p>{{ error }}</p>
        <ion-button @click="fetchData" color="primary" fill="outline">
          Coba Lagi
        </ion-button>
      </div>

      <!-- Data Table/Grid -->
      <div v-else class="crypto-table">
        <!-- Header Row -->
        <div class="table-header">
          <div class="cell rank">#</div>
          <div class="cell name">Name</div>
          <div class="cell symbol">Symbol</div>
          <div class="cell price">Price (USD)</div>
        </div>

        <!-- Data Rows -->
        <div
          v-for="coin in filteredCoins"
          :key="coin.id"
          class="table-row"
          @click="goToDetail(coin)"
        >
          <div class="cell rank">{{ coin.rank }}</div>
          <div class="cell name">
            <div class="coin-info">
              <ion-icon :name="getCoinIcon(coin.symbol)" class="coin-icon" />
              <span class="coin-name">{{ coin.name }}</span>
            </div>
          </div>
          <div class="cell symbol">{{ coin.symbol }}</div>
          <div class="cell price">{{ formatPrice(coin.price_usd) }}</div>
        </div>

        <!-- Empty State -->
        <div v-if="filteredCoins.length === 0" class="empty-state">
          <ion-icon name="search-outline" size="large" />
          <p>Tidak ada data yang cocok</p>
        </div>
      </div>

      <!-- Stats Footer -->
      <div v-if="!loading && coins.length > 0" class="stats-footer">
        <span>Menampilkan {{ filteredCoins.length }} dari {{ coins.length }} crypto</span>
        <span class="last-updated">Terakhir update: {{ formatLastUpdated }}</span>
      </div>
    </ion-content>

    <!-- Pull to Refresh -->
    <ion-refresher slot="fixed" @ionRefresh="handleRefresh($event)">
      <ion-refresher-content
        pulling-icon="arrow-down-outline"
        pulling-text="Tarik untuk refresh"
        refreshing-spinner="circles"
        refreshing-text="Memuat data...">
      </ion-refresher-content>
    </ion-refresher>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import { IonContent, IonHeader, IonTitle, IonToolbar, IonPage, IonSearchbar, IonButton, IonIcon, IonRefresher, IonRefresherContent } from '@ionic/vue';
import { refresh, alertCircleOutline, searchOutline, arrowDownOutline, logoBitcoin, logoEthereum, cardOutline } from 'ionicons/icons';

const API_URL = 'https://api.coinlore.net/api/tickers/';

interface Coin {
  id: string;
  symbol: string;
  name: string;
  nameid: string;
  rank: number;
  price_usd: string;
  percent_change_24h: string;
  percent_change_1h: string;
  percent_change_7d: string;
  price_btc: string;
  market_cap_usd: string;
  volume24: number;
  volume24a: number;
  csupply: string;
  tsupply: string;
  msupply: string;
}

const coins = ref<Coin[]>([]);
const loading = ref(false);
const error = ref<string | null>(null);
const searchQuery = ref('');
const lastFetched = ref<Date | null>(null);

const filteredCoins = computed(() => {
  if (!searchQuery.value) return coins.value;
  const q = searchQuery.value.toLowerCase();
  return coins.value.filter(c =>
    c.name.toLowerCase().includes(q) ||
    c.symbol.toLowerCase().includes(q)
  );
});

const formatLastUpdated = computed(() => {
  if (!lastFetched.value) return '-';
  return lastFetched.value.toLocaleString('id-ID', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
  });
});

const formatPrice = (price: string) => {
  const num = parseFloat(price);
  if (num >= 1) return '$' + num.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
  return '$' + num.toFixed(6);
};

const getCoinIcon = (symbol: string) => {
  const icons: Record<string, string> = {
    BTC: 'logo-bitcoin',
    ETH: 'logo-ethereum',
    USDT: 'card-outline',
    USDC: 'card-outline',
    BNB: 'logo-bitcoin',
  };
  return icons[symbol.toUpperCase()] || 'card-outline';
};

const fetchData = async () => {
  loading.value = true;
  error.value = null;
  try {
    const res = await fetch(API_URL);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    coins.value = data.data || [];
    lastFetched.value = new Date();
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Gagal memuat data';
    console.error(e);
  } finally {
    loading.value = false;
  }
};

const handleRefresh = (event: CustomEvent) => {
  fetchData().finally(() => {
    (event.target as HTMLIonRefresherElement).complete();
  });
};

const goToDetail = (coin: Coin) => {
  // Navigate to detail page if needed
  console.log('Navigate to detail:', coin);
};

onMounted(() => {
  fetchData();
});
</script>

<style scoped>
/* Searchbar spacing */
.mb-4 {
  margin-bottom: 1rem;
}

/* Skeleton Loading */
.skeleton-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.skeleton-row {
  display: grid;
  grid-template-columns: 50px 1fr 80px 120px;
  gap: 12px;
  padding: 12px 16px;
  background: var(--ion-background-color);
  border-radius: 8px;
  border: 1px solid var(--ion-border-color, #e0e0e0);
}

.skeleton-cell {
  height: 20px;
  border-radius: 4px;
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.skeleton-cell.rank { width: 40px; }
.skeleton-cell.name { width: 100%; }
.skeleton-cell.symbol { width: 70px; }
.skeleton-cell.price { width: 110px; }

/* Error State */
.error-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 200px;
  gap: 12px;
  color: var(--ion-color-danger);
  text-align: center;
  padding: 24px;
}

/* Table/Grid Layout */
.crypto-table {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.table-header {
  display: grid;
  grid-template-columns: 50px 1fr 80px 120px;
  gap: 12px;
  padding: 12px 16px;
  font-weight: 600;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  color: var(--ion-color-medium, #666);
  background: var(--ion-color-light, #f5f5f5);
  border-radius: 8px 8px 0 0;
  position: sticky;
  top: 0;
  z-index: 10;
}

.table-row {
  display: grid;
  grid-template-columns: 50px 1fr 80px 120px;
  gap: 12px;
  padding: 14px 16px;
  background: var(--ion-background-color, #fff);
  border-bottom: 1px solid var(--ion-border-color, #eee);
  transition: background 0.15s;
  cursor: pointer;
}

.table-row:last-child {
  border-bottom: none;
  border-radius: 0 0 8px 8px;
}

.table-row:active {
  background: var(--ion-color-light, #f8f8f8);
}

.cell {
  display: flex;
  align-items: center;
  font-size: 0.9rem;
}

.cell.rank {
  font-weight: 600;
  color: var(--ion-color-medium, #666);
  min-width: 40px;
}

.cell.name {
  min-width: 0;
}

.coin-info {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 0;
}

.coin-icon {
  font-size: 1.2rem;
  color: var(--ion-color-primary);
  flex-shrink: 0;
}

.coin-name {
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.cell.symbol {
  font-family: monospace;
  font-weight: 600;
  color: var(--ion-color-dark, #333);
  text-transform: uppercase;
  font-size: 0.8rem;
}

.cell.price {
  font-family: monospace;
  font-weight: 500;
  text-align: right;
  color: var(--ion-color-dark, #333);
  white-space: nowrap;
}

/* Empty State */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 48px 24px;
  color: var(--ion-color-medium);
  gap: 12px;
}

/* Stats Footer */
.stats-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 16px;
  padding: 12px 16px;
  font-size: 0.75rem;
  color: var(--ion-color-medium);
  background: var(--ion-color-light, #f5f5f5);
  border-radius: 8px;
}

.last-updated {
  font-family: monospace;
}

/* Responsive adjustments */
@media (max-width: 480px) {
  .table-header,
  .table-row,
  .skeleton-row {
    grid-template-columns: 40px 1fr 70px 100px;
    gap: 8px;
    padding: 10px 12px;
  }

  .cell.price {
    font-size: 0.8rem;
  }

  .coin-icon {
    font-size: 1rem;
  }
}
</style>
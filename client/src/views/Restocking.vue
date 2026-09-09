<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budgetLabel') }}</h3>
          <span class="budget-value">{{ currencySymbol }}{{ budget.toLocaleString() }}</span>
        </div>
        <input
          v-model.number="budget"
          type="range"
          min="0"
          max="10000"
          step="100"
          class="budget-slider"
        />
      </div>

      <div v-if="successMessage" class="success-message">{{ successMessage }}</div>
      <div v-if="submitError" class="error">{{ submitError }}</div>

      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendedItems') }}</h3>
        </div>

        <div v-if="recommendations.length === 0" class="empty-state">
          {{ t('restocking.noRecommendations') }}
        </div>
        <template v-else>
          <div class="table-container">
            <table class="restocking-table">
              <thead>
                <tr>
                  <th class="col-sku">{{ t('restocking.table.sku') }}</th>
                  <th class="col-item-name">{{ t('restocking.table.itemName') }}</th>
                  <th class="col-demand">{{ t('restocking.table.currentDemand') }}</th>
                  <th class="col-demand">{{ t('restocking.table.forecastedDemand') }}</th>
                  <th class="col-demand">{{ t('restocking.table.demandGap') }}</th>
                  <th class="col-cost">{{ t('restocking.table.unitCost') }}</th>
                  <th class="col-quantity">{{ t('restocking.table.quantity') }}</th>
                  <th class="col-cost">{{ t('restocking.table.subtotal') }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in recommendations" :key="item.item_sku">
                  <td class="col-sku"><strong>{{ item.item_sku }}</strong></td>
                  <td class="col-item-name">{{ item.item_name }}</td>
                  <td class="col-demand">{{ item.current_demand }}</td>
                  <td class="col-demand">{{ item.forecasted_demand }}</td>
                  <td class="col-demand">{{ item.demandGap }}</td>
                  <td class="col-cost">{{ currencySymbol }}{{ item.unit_cost.toLocaleString() }}</td>
                  <td class="col-quantity">{{ item.quantity }}</td>
                  <td class="col-cost"><strong>{{ currencySymbol }}{{ item.subtotal.toLocaleString() }}</strong></td>
                </tr>
              </tbody>
            </table>
          </div>

          <div class="summary-bar">
            <div class="summary-item">
              <span class="summary-label">{{ t('restocking.itemCount', { count: recommendations.length }) }}</span>
            </div>
            <div class="summary-item">
              <span class="summary-label">{{ t('restocking.totalCost') }}</span>
              <span class="summary-value">{{ currencySymbol }}{{ totalCost.toLocaleString() }}</span>
            </div>
            <div class="summary-item">
              <span class="summary-label">{{ t('restocking.remainingBudget') }}</span>
              <span class="summary-value">{{ currencySymbol }}{{ remainingBudget.toLocaleString() }}</span>
            </div>
          </div>
        </template>

        <button
          class="place-order-btn"
          :disabled="submitting || recommendations.length === 0"
          @click="placeOrder"
        >
          {{ submitting ? t('restocking.placingOrder') : t('restocking.placeOrder') }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency } = useI18n()

    const currencySymbol = computed(() => {
      return currentCurrency.value === 'JPY' ? '¥' : '$'
    })

    const loading = ref(true)
    const error = ref(null)
    const forecasts = ref([])

    const budget = ref(2500)
    const submitting = ref(false)
    const successMessage = ref(null)
    const submitError = ref(null)

    const loadForecasts = async () => {
      try {
        loading.value = true
        error.value = null
        forecasts.value = await api.getDemandForecasts()
      } catch (err) {
        error.value = 'Failed to load demand forecasts: ' + err.message
      } finally {
        loading.value = false
      }
    }

    const recommendations = computed(() => {
      const candidates = forecasts.value
        .map(f => ({
          item_sku: f.item_sku,
          item_name: f.item_name,
          current_demand: f.current_demand,
          forecasted_demand: f.forecasted_demand,
          demandGap: f.forecasted_demand - f.current_demand,
          unit_cost: f.unit_cost
        }))
        .filter(f => f.demandGap > 0)
        .sort((a, b) => b.demandGap - a.demandGap)

      const result = []
      let remainingBudget = budget.value

      for (const item of candidates) {
        const quantity = Math.min(item.demandGap, Math.floor(remainingBudget / item.unit_cost))
        if (quantity <= 0) continue

        const subtotal = quantity * item.unit_cost
        result.push({ ...item, quantity, subtotal })
        remainingBudget -= subtotal
      }

      return result
    })

    const totalCost = computed(() => {
      return recommendations.value.reduce((sum, item) => sum + item.subtotal, 0)
    })

    const remainingBudget = computed(() => budget.value - totalCost.value)

    const placeOrder = async () => {
      submitting.value = true
      successMessage.value = null
      submitError.value = null

      try {
        const order = await api.createRestockingOrder({
          budget: budget.value,
          items: recommendations.value.map(r => ({
            item_sku: r.item_sku,
            item_name: r.item_name,
            quantity: r.quantity,
            unit_cost: r.unit_cost
          }))
        })

        successMessage.value = t('restocking.orderPlaced', {
          orderNumber: order.order_number,
          days: order.lead_time_days
        })

        await loadForecasts()
      } catch (err) {
        submitError.value = t('restocking.orderFailed')
      } finally {
        submitting.value = false
      }
    }

    onMounted(loadForecasts)

    return {
      t,
      currencySymbol,
      loading,
      error,
      budget,
      recommendations,
      totalCost,
      remainingBudget,
      submitting,
      successMessage,
      submitError,
      placeOrder
    }
  }
}
</script>

<style scoped>
.card-header {
  align-items: center;
}

.budget-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
}

.budget-slider {
  width: 100%;
  accent-color: #2563eb;
}

.success-message {
  background: #ecfdf5;
  border: 1px solid #a7f3d0;
  color: #065f46;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}

.empty-state {
  text-align: center;
  padding: 2rem;
  color: #64748b;
  font-size: 0.938rem;
}

.restocking-table {
  table-layout: fixed;
  width: 100%;
}

.col-sku {
  width: 110px;
}

.col-item-name {
  width: 200px;
}

.col-demand {
  width: 130px;
}

.col-cost {
  width: 110px;
}

.col-quantity {
  width: 100px;
}

.summary-bar {
  display: flex;
  gap: 2rem;
  padding: 1rem 0.75rem;
  margin-top: 0.5rem;
  border-top: 1px solid #e2e8f0;
}

.summary-item {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.summary-label {
  font-size: 0.813rem;
  color: #64748b;
  font-weight: 600;
}

.summary-value {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
}

.place-order-btn {
  margin-top: 1.25rem;
  padding: 0.625rem 1.5rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease;
}

.place-order-btn:hover:not(:disabled) {
  background: #1d4ed8;
}

.place-order-btn:disabled {
  background: #cbd5e1;
  cursor: not-allowed;
}
</style>

<template>
  <AppHeaders />
  <div class="container">
    <AppBalance :total="total" />
    <IncomeExpenses :income="income" :expenses="expenses" />
    <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
import AppHeaders from './components/AppHeaders.vue';
import AppBalance from './components/AppBalance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

import { computed, ref, onMounted, watch } from 'vue';
import { useToast } from 'vue-toastification';

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'));

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

// get total
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => acc + transaction.amount, 0);
});

// get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => acc + transaction.amount, 0)
    .toFixed(2);
});

// get expense
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => acc + transaction.amount, 0)
    .toFixed(2);
});

// add new transaction
const handleTransactionSubmitted = (transaction) => {
  transactions.value.push({
    id: transactions.value.length,
    ...transaction
  });

  toast.success('Transaction added');
};

// deleted transaction
const handleTransactionDeleted = (id) => {
  const newTransaction = transactions.value.filter((transaction) => transaction.id !== id);
  transactions.value = newTransaction;
  toast.success('Transaction deleted');
};

// save transaction to local storage
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value));
};

// watch transactions
watch(
  () => transactions.value.length,
  () => {
    console.log('transactions');
    saveTransactionsToLocalStorage();
  }
);
</script>

<template>
  <Header/>
  <div class="container">
    <Balance :total="+total"/>
    <IncomeExpenses :income="+income" :expenses="+expenses" />
    <TransactionList @transactionDeleted="handleTransactionDeleted" :transactions="transactions"/>
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>
import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

import {useToast} from 'vue-toastification';
import { ref, computed, onMounted } from 'vue';

const toast = useToast();
const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'));

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

// Get total
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0)
})

// Get income
const income = computed(() => {
  return transactions.value
    .filter(transaction => transaction.amount > 0)
    .reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0).toFixed(2);
})

// Get expenses
const expenses = computed(() => {
  return transactions.value
    .filter(transaction => transaction.amount < 0)
    .reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0).toFixed(2);
})

// Handles the submission of a transaction and updates the transactions list with the new transaction.
const handleTransactionSubmitted = (transactionData) => {
  transactions.value.push({
    id: generateTransactionId(),
    text: transactionData.text,
    amount: +transactionData.amount
  });

  saveToLocalStorage();

  toast.success('Transaction added');
}

// Generate a transaction ID using a random number generator.
const generateTransactionId = () => {
  return Math.floor(Math.random() * 1000000000)
}

// Handles the deletion of a transaction with the given id from the transactions array.
const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter(transaction => transaction.id !== id)

  saveToLocalStorage();

  toast.success('Transaction removed');
}

// Saves the 'transactions' value to the local storage as a JSON string.
const saveToLocalStorage = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value))
}
</script>
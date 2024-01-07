<template>
  <div>
    <Header />

    <div class="container">
      <Balance :total="+total"/>
      <IncomeExpenses :income="+income" :expenses="+expenses"/>
      <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
      <AddTransaction  @transactionEmitted="handleTransactionSubmitted"/>
    </div>
  </div>
</template>

<script setup>
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpenses from "./components/IncomeExpenses.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";
import { computed, ref, onMounted } from "vue";

import {useToast} from 'vue-toastification'

const toast = useToast();
const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'));

  if(savedTransactions){
    transactions.value = savedTransactions;
  }
});
// Get Total 
const total = computed(()=>{
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  },0)
});

// Get Income 
const income = computed(()=>{
  return transactions.value.filter((transaction)=> transaction.amount > 0)
  .reduce((acc, transaction) => {
    return acc + transaction.amount;
  },0).toFixed(2);
});

// Get Expense 
const expenses = computed(()=>{
  return transactions.value.filter((transaction)=> transaction.amount < 0)
  .reduce((acc, transaction) => {
    return acc + transaction.amount;
  },0).toFixed(2);
});

// Add Transaction Data 
const handleTransactionSubmitted = (transactionData) => {
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount
  });

  saveTransactionsToLocalStorage();
  toast.success("Transaction added!");
};

// Generate Unique Id
const generateUniqueId = () => {
  return Math.floor(Math.random() * 10000);
};

//Delete Transaction
const handleTransactionDeleted = (id) =>{
  transactions.value = transactions.value.filter((transactions) => transactions.id !== id);

  saveTransactionsToLocalStorage();
  toast.success('Transaction Deleted');
};

//Save Transactions to Localstorage
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem('transactions',JSON.stringify(transactions.value));
}
</script>
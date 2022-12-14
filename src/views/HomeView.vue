<script setup>
import ExpressionGroup from "../components/ExpressionGroup.vue";
import { mapStores } from "pinia";
import {
  useExpressionsStore,
  EXPRESSIONS_LIMIT,
} from "../stores/expressions/expressions.js";
import { useTableStore } from "../stores/table/table";
import SelectButton from "primevue/selectbutton";
import { v4 as generateRandomUUID } from "uuid";
import { AND, OR } from "../stores/expressions/expressions.js";
import { ExpressionModel } from "../model/ExpressionModel.js";
import Message from "primevue/message";
import ProgressSpinner from 'primevue/progressspinner';
</script>

<template>
  <main
    class="
      w-full
      min-h-screen
      flex flex-col
      p-10
      pr-60
      h-full
      overflow-y-scroll
    "
    v-if="loaded"
  >
    <Modal> </Modal>
    <div v-for="(exp, index) in expressionsStore.expressions" :key="index">
      <div
        v-if="exp == AND || exp == OR"
        class="
          font-semibold
          bg-indigo-900
          rounded-3xl
          w-20
          h-10
          flex
          items-center
          text-white
          justify-center
          relative
          mt-3
          mb-3
          z-10
          relative
        "
      >
        <p class="z-20 text-sm">{{ exp }}</p>
        <div class="bg-indigo-900 w-[1.5px] h-[65px] absolute z-0"></div>
      </div>
      <ExpressionGroup
        v-else
        class="z-index10 relative mr-20"
        :expressionGroupIndex="index"
      >
      </ExpressionGroup>
    </div>

    <div class="flex mt-10 mb-12 mr-20">
      <div
        class="
          bg-indigo-900
          w-fit
          rounded-3xl
          shadow-md
          flex
          items-center
          p-3
          hover:bg-indigo-700
          cursor-pointer
        "
        :class="disable"
        v-on:click="addExpressionGroup"
      >
        <img src="/add.png" class="w-6" draggable="false" />
        <p class="font-semibold text-sm ml-1 text-white mr-1">
          Add New Condition Group
        </p>
      </div>

      <SelectButton
        v-model="operator"
        :options="options"
        aria-labelledby="single"
        class="ml-[15px] mt-auto mb-auto mr-4"
      />
      <div class="ml-auto flex">
        <div
          v-on:click="clear"
          class="
            flex
            items-center
            mr-4
            bg-white
            text-gray-700
            hover:bg-indigo-50/25
            pl-4
            pr-4
            rounded-xl
            shadow-md
            font-semibold
            cursor-pointer
          "
        >
          Clear All
          <p class="ml-1 text-rose-400 font-bold text-lg"><strong>!</strong></p>
        </div>
        <button
          v-on:click="buildRule"
          class="
            mr-4
            bg-indigo-900
            hover:bg-indigo-700
            text-white
            pl-4
            pr-4
            rounded-xl
            shadow-md
            font-semibold
            cursor-pointer
          "
        >
          Save Rule
        </button>
      </div>
    </div>
    <div
      class="relative bg-rose-200 w-50 h-16 mr-20 rounded-xl flex items-center  mb-4   shadow-lg"
      :class="hideRuleError"
    >
      <div class="bg-rose-600 w-2 h-full rounded-l-xl"></div>
      <p class="text-rose-600 text-md ml-10">This is not a valid rule</p>
 
    </div>

    <div
      class="relative bg-green-100 w-50 h-16 mr-20 rounded-xl flex items-center  mb-4   shadow-lg"
      :class="ruleSavedHide"
    >
      <div class="bg-green-500 w-2 h-full rounded-l-xl"></div>
      <p class="text-green-600 text-md ml-10">Rule successfully saved, you can use it in the rules section</p>
 
    </div>
  </main>
  <div v-else class="h-full w-full flex flex-col items-center justify-center">
    <ProgressSpinner/>
  </div>
</template>

<script>
export default {
  props:{
    ruleSavedHide: String
  },
  data() {
    return {
      rule: [],
      operator: AND,
      backUpOperator:AND,
      options: [AND, OR],
      expressions: ["ooo"],
      hideRuleError: "hidden",
      hideRuleSaved:'hidden',
      loaded:false
    };
  },
  methods: {
    addExpressionGroup() {
      if (this.expressionsStore.expressionsCounter < EXPRESSIONS_LIMIT) {
        this.expressionsStore.addExpression(
          this.operator,
          generateRandomUUID(),
          this.expressionsStore.expressions.length
        );
        this.expressionsStore.addExpression(
          new ExpressionModel(),
          generateRandomUUID(),
          this.expressionsStore.expressions.length
        );
      }
    },
    buildRule() {
   
      if (this.expressionsStore.isValidRule())
        this.$emit('openModal');
        else this.hideRuleError = '';

        setTimeout(()=>{this.hideRuleError = 'hidden';}, 3000)
    },
   
    clear() {
      this.expressionsStore.clear();
      this.expressionsStore.addExpression(
        new ExpressionModel(),
        generateRandomUUID(),
        0
      );
    },
  },
  mounted() {

    this.tableStore.laodColumns().then(data=>{
      this.loaded=true;
    });
    this.expressionsStore.addExpression(
      new ExpressionModel(),
      generateRandomUUID(),
      0
    );
  },
  computed: {
    ...mapStores(useExpressionsStore, useTableStore),
    disable() {
      let _class =
        this.expressionsStore.expressionsCounter >= EXPRESSIONS_LIMIT
          ? "opacity-40 hover:bg-indigo-900 cursor-not-allowed"
          : "";
      return _class;
    },
  },
  watch: {
    operator() {
      if (this.operator == null) {
        this.operator = this.backUpOperator;
      } else {
        this.backUpOperator = this.operator;
      }
    },
  },
};
</script>

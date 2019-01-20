<template>
  <div class="vqb-rule font-secondary">
    <div class="flex flex-wrap content-around items-center py-1">
      <label class="text-lg font-normal pr-4">{{ rule.label }}</label>

      <select v-if="typeof rule.operands !== 'undefined'" v-model="query.selectedOperand" class="bg-grey-lighter rounded transition h-6 px-4 mx-2">
        <option v-for="(operand, index) in rule.operands" :key="index">{{ operand }}</option>
      </select>

      <select v-if="! isMultipleChoice" v-model="query.selectedOperator" class="bg-grey-lighter rounded transition h-6 px-4 mx-2">
        <option v-for="(operator, index) in rule.operators" v-bind:value="operator" :key="index">
          {{ operator }}
        </option>
      </select>

      <input class="bg-grey-lighter rounded appearance-none my-1 py-1 px-4" v-if="rule.inputType === 'text'" type="text" v-model="query.value" :placeholder="labels.textInputPlaceholder">
      <input class="bg-grey-lighter rounded appearance-none my-1 py-1 px-4" v-if="rule.inputType === 'number'" type="number" v-model="query.value">

      <template v-if="isCustomComponent">
        <component :value="query.value" @input="updateQuery" :is="rule.component"></component>
      </template>

      <div class="checkbox" v-if="rule.inputType === 'checkbox'">
        <label class="px-2 font-normal" v-for="(choice, index) in rule.choices" :key="index">
          <input type="checkbox" :value="choice.value" v-model="query.value"> {{ choice.label }}
        </label>
      </div>

      <div class="radio" v-if="rule.inputType === 'radio'">
        <label class="px-2 font-normal" v-for="(choice, index) in rule.choices" :key="index">
          <input type="radio" :value="choice.value" v-model="query.value"> {{ choice.label }}
        </label>
      </div>

      <select
        v-if="rule.inputType === 'select'"
        class="bg-grey-lighter rounded transition h-6 px-4 mx-2"
        :multiple="rule.type === 'multi-select'"
        v-model="query.value">

        <template v-for="(option, option_key) in selectOptions">
          <option v-if="!Array.isArray(option)" :value="option.value" :key="option_key">
            {{ option.label }}
          </option>
          <optgroup v-if="Array.isArray(option)" :label="option_key" :key="option_key">
            <option v-for="(sub_option, index) in option" :key="index" :value="sub_option.value">{{ sub_option.label }}</option>
          </optgroup>
        </template>

      </select>

      <button type="button" class="bg-primary rounded transition text-xl text-white h-6 px-2 mx-2" @click="remove" v-html="labels.removeRule">X</button>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import deepClone from "../utilities.js";

export default {
  name: "query-builder-rule",

  props: ["query", "index", "rule", "styled", "labels"],

  beforeMount() {
    if (this.rule.type === "custom-component") {
      this.$options.components[this.id] = this.rule.component;
    }
  },

  methods: {
    remove: function() {
      this.$emit("child-deletion-requested", this.index);
    },
    updateQuery(value) {
      let updated_query = deepClone(this.query);
      updated_query.value = value;
      this.$emit("update:query", updated_query);
    }
  },

  computed: {
    isMultipleChoice() {
      return ["radio", "checkbox", "select"].indexOf(this.rule.inputType) >= 0;
    },

    isCustomComponent() {
      return this.rule.type === "custom-component";
    },

    selectOptions() {
      if (typeof this.rule.choices === "undefined") {
        return {};
      }

      return this.rule.choices.reduce(function(groups, item, index) {
        let key = item["group"];
        if (typeof key !== "undefined") {
          groups[key] = groups[key] || [];
          groups[key].push(item);
        } else {
          groups[index] = item;
        }

        return groups;
      }, {});
    }
  },

  mounted() {
    let updated_query = deepClone(this.query);

    // Set a default value for these types if one isn't provided already
    if (this.query.value === null) {
      if (this.rule.inputType === "checkbox") {
        updated_query.value = [];
      }
      if (this.rule.type === "select") {
        updated_query.value = this.rule.choices[0].value;
      }
      if (this.rule.type === "custom-component") {
        updated_query.value = this.rule.default || null;
      }

      this.$emit("update:query", updated_query);
    }
  }
};
</script>

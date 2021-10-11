<template>
  <n-button
    text
    type="primary"
    @click="showModal = true"
  >
    修改展示的逻辑符号
  </n-button>
  <p>
    <n-ellipsis
      expand-trigger="click"
      :line-clamp="1"
      :tooltip="false"
    >
      主合取范式 =
      <span v-if="pnf.pcnfSub.length">
        π
        <sub>{{ pnf.pcnfSub }}</sub> =
      </span>
      {{ pnf.pcnf }}
    </n-ellipsis>
  </p>
  <p>
    <n-ellipsis
      expand-trigger="click"
      :line-clamp="1"
      :tooltip="false"
    >
      主析取范式 =
      <span v-if="pnf.pdnfSub.length">
        Σ
        <sub>{{ pnf.pdnfSub }}</sub> =
      </span>
      {{ pnf.pdnf }}
    </n-ellipsis>
  </p>
  <p>
    <n-ellipsis
      expand-trigger="click"
      :line-clamp="1"
      :tooltip="false"
    >
      最简合取范式 =
      {{ mnf.cnf }}
    </n-ellipsis>
  </p>
  <p>
    <n-ellipsis
      expand-trigger="click"
      :line-clamp="1"
      :tooltip="false"
    >
      最简析取范式 =
      {{ mnf.dnf }}
    </n-ellipsis>
  </p>
  <n-modal
    v-model:show="showModal"
    preset="dialog"
    title="修改展示的逻辑符号"
  >
    <n-form
      ref="formRef"
      inline
      :label-width="80"
      :model="symbols"
      :rules="rules"
    >
      <n-form-item
        label="逻辑与"
        path="conjunction"
      >
        <n-input
          v-model:value="symbols.conjunction"
          placeholder="请输入逻辑与符号"
        />
      </n-form-item>
      <n-form-item
        label="逻辑或"
        path="disjunction"
      >
        <n-input
          v-model:value="symbols.disjunction"
          placeholder="请输入逻辑或符号"
        />
      </n-form-item>
      <n-form-item
        label="逻辑取反"
        path="not"
      >
        <n-input
          v-model:value="symbols.not"
          placeholder="请输入逻辑取反符号"
        />
      </n-form-item>
    </n-form>
    <template #action>
      <n-button
        type="primary"
        @click="submitCallback"
      >
        保存
      </n-button>
    </template>
  </n-modal>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';

import {
  NEllipsis, NButton, NModal, NForm, NFormItem, NInput,
} from 'naive-ui';
import QuineMcCluskey from '~/core/QuineMcCluskey';
import ICostomSymbol from '~/types/costomSymbol';

const props = defineProps<{
  atoms: string[],
  truths: boolean[],
}>();

const formRef = ref();

const rules = {
  conjunction: { required: true, trigger: 'blur', message: '请输入逻辑与符号' },
  disjunction: { required: true, trigger: 'blur', message: '请输入逻辑或符号' },
  not: { required: true, trigger: 'blur', message: '请输入逻辑取反符号' },
};

const SYMBOL_KEY = 'symbols';

const defaultSymbol = {
  conjunction: '∧',
  disjunction: '∨',
  not: '¬',
};

const localSymbol = localStorage.getItem(SYMBOL_KEY);

const symbols = ref<ICostomSymbol>(localSymbol ? JSON.parse(localSymbol) : defaultSymbol);

const pnf = computed(() => {
  const pcnfParts: string[] = [];
  const pdnfParts: string[] = [];
  const pcnfNums: number[] = [];
  const pdnfNums: number[] = [];

  props.truths.forEach((truth, i) => {
    const parts: string[] = [];
    props.atoms.forEach((atom, j) => {
      if (truth === !!((i >> (props.atoms.length - 1 - j)) & 1)) {
        parts.push(atom);
      } else {
        parts.push(`${symbols.value.not}${atom}`);
      }
    });
    if (truth) {
      pdnfNums.push(i);
      pdnfParts.push(`(${parts.join(` ${symbols.value.conjunction} `)})`);
    } else {
      pcnfNums.push(i);
      pcnfParts.push(`(${parts.join(` ${symbols.value.disjunction} `)})`);
    }
  });

  return {
    pcnf: pcnfParts.length ? pcnfParts.join(` ${symbols.value.conjunction} `) : 'T',
    pdnf: pdnfParts.length ? pdnfParts.join(` ${symbols.value.disjunction} `) : 'F',
    pcnfSub: pcnfNums.join(', '),
    pdnfSub: pdnfNums.join(', '),
    pdnfNums,
    pcnfNums,
  };
});

const mnf = computed(() => {
  const cnf = new QuineMcCluskey(props.atoms.join(''), pnf.value.pcnfNums, symbols.value, [], true).getFunction();
  const dnf = new QuineMcCluskey(props.atoms.join(''), pnf.value.pdnfNums, symbols.value).getFunction();

  return {
    dnf,
    cnf,
  };
});

const showModal = ref(false);

function submitCallback() {
  formRef.value.validate((errors: any) => {
    if (!errors) {
      localStorage.setItem(SYMBOL_KEY, JSON.stringify(symbols.value));
      showModal.value = false;
    } else {
      showModal.value = true;
    }
  });
}

</script>

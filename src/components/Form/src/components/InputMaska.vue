<script setup>
  import { vMaska } from 'maska/vue';
  import { Input } from 'ant-design-vue';

  import { ref, watch } from 'vue';

  const props = defineProps({
    value: String,
  });
  const testModel = ref(props.value);

  const emits = defineEmits(['change']);
  function test(e) {
    if (e.detail.unmasked) {
      emits('change', +e?.detail?.unmasked);
    }
  }
  function test2(e) {
    if (!e) {
      emits('change', e);
    }
  }
  watch(
    () => props.value,
    (value) => {
      testModel.value = +value;
    },
  );
</script>

<template>
  <Input
    @update:value="test2"
    @maska="test"
    v-maska="{ number: { locale: 'eu', fraction: 2 } }"
    v-model:value="testModel"
  />
</template>

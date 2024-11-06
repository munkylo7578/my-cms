<template>
  <TreeSelect
    v-model:value="state"
    style="width: 100%"
    :tree-data="computedTreeData"
    tree-checkable
    allow-clear
    :show-checked-strategy="SHOW_PARENT"
    placeholder="Please select"
    tree-node-filter-prop="label"
  />
</template>
<script lang="ts" setup>
  import { computed, ref, watch } from 'vue';
  import type { TreeSelectProps } from 'ant-design-vue';
  import { TreeSelect } from 'ant-design-vue';
  import { useRuleFormItem } from '@/hooks/component/useFormItem';
  const SHOW_PARENT = TreeSelect.SHOW_CHILD;
  const props = defineProps({
    value: Array as PropType<string[]>,
    treeData: Array as PropType<TreeSelectProps['treeData']>,
  });
  const emits = defineEmits(['change', 'update:value']);
  const [state, setState] = useRuleFormItem(props, 'value', 'change');
  const computedTreeData = computed(() => {
    const temp = props.treeData?.map((item) => {
      return {
        label: item.label,
        value: 'parent-' + item.value,
        children: item.children?.map((childItem) => ({
          label: childItem.label,
          value: 'child-' + childItem.value,
        })),
      };
    });
    debugger;
    return temp;
  });
</script>

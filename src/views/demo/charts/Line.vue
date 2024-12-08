<template>
  <div ref="chartRef" :style="{ height, width }"></div>
</template>
<script lang="ts" setup>
  import { PropType, ref, Ref, onMounted } from 'vue';
  import { useECharts } from '@/hooks/web/useECharts';
  import { getLineData } from './data';
  import 'reflect-metadata';

  function GraphQLField(): PropertyDecorator {
    return (target, propertyKey) => {
      const fields = Reflect.getMetadata('graphql:fields', target) || [];
      fields.push(propertyKey.toString());
      Reflect.defineMetadata('graphql:fields', fields, target);
    };
  }
  function GraphQLNested(type: () => any): PropertyDecorator {
    return (target, propertyKey) => {
      const nestedFields = Reflect.getMetadata('graphql:nestedFields', target) || {};
      nestedFields[propertyKey.toString()] = type;
      Reflect.defineMetadata('graphql:nestedFields', nestedFields, target);
    };
  }
  function generateFields(classInstance: Object): string {
    const fields = Reflect.getMetadata('graphql:fields', classInstance) || [];
    const nestedFields = Reflect.getMetadata('graphql:nestedFields', classInstance) || {};

    const fieldsString = fields.map((field: string) => field).join(' ');

    const nestedFieldsString = Object.entries(nestedFields)
      .map(([key, nestedType]: [string, any]) => {
        const NestedClass = nestedType();
        const nestedInstance = new NestedClass();
        return `${key} { ${generateFields(nestedInstance)} }`;
      })
      .join(' ');

    return `${fieldsString} ${nestedFieldsString}`.trim();
  }
  class Test {
    @GraphQLField()
    hello!: string;
  }
  class Address {
    @GraphQLField()
    street!: string;

    @GraphQLField()
    city?: string;

    @GraphQLField()
    country!: string;

    @GraphQLNested(() => Test)
    test?: Test;
  }
  class User {
    @GraphQLField()
    id!: string;

    @GraphQLField()
    name!: string;

    @GraphQLNested(() => Address)
    addresses!: Address[];
  }

  function generateGraphQLQuery(
    classInstance: Object,
    operation: 'query' | 'mutation',
    operationName: string,
  ): string {
    const className = classInstance.constructor.name.toLowerCase();
    const fieldsString = generateFields(classInstance);

    return `
    ${operation} ${operationName} {
      ${className} {
        ${fieldsString}
      }
    }
  `;
  }
  const user = new User();
  const query = generateGraphQLQuery(user, 'query', 'GetUser');
  console.log(query, 'hehehee');

  defineProps({
    width: {
      type: String as PropType<string>,
      default: '100%',
    },
    height: {
      type: String as PropType<string>,
      default: 'calc(100vh - 78px)',
    },
  });

  const chartRef = ref<HTMLDivElement | null>(null);
  const { setOptions, echarts } = useECharts(chartRef as Ref<HTMLDivElement>);
  const { barData, lineData, category } = getLineData;

  onMounted(() => {
    setOptions({
      backgroundColor: '#0f375f',
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'shadow',
          label: {
            show: true,
            backgroundColor: '#333',
          },
        },
      },
      legend: {
        data: ['line', 'bar'],
        textStyle: {
          color: '#ccc',
        },
      },
      xAxis: {
        data: category,
        axisLine: {
          lineStyle: {
            color: '#ccc',
          },
        },
      },
      yAxis: {
        splitLine: { show: false },
        axisLine: {
          lineStyle: {
            color: '#ccc',
          },
        },
      },
      series: [
        {
          name: 'line',
          type: 'line',
          smooth: true,
          showAllSymbol: 'auto',
          symbol: 'emptyCircle',
          symbolSize: 15,
          data: lineData,
        },
        {
          name: 'bar',
          type: 'bar',
          barWidth: 10,
          itemStyle: {
            borderRadius: 5,
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: '#14c8d4' },
              { offset: 1, color: '#43eec6' },
            ]),
          },
          data: barData,
        },
        {
          name: 'line',
          type: 'bar',
          barGap: '-100%',
          barWidth: 10,
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: 'rgba(20,200,212,0.5)' },
              { offset: 0.2, color: 'rgba(20,200,212,0.2)' },
              { offset: 1, color: 'rgba(20,200,212,0)' },
            ]),
          },
          z: -12,
          data: lineData,
        },
        {
          name: 'dotted',
          type: 'pictorialBar',
          symbol: 'rect',
          itemStyle: {
            color: '#0f375f',
          },
          symbolRepeat: true,
          symbolSize: [12, 4],
          symbolMargin: 1,
          z: -10,
          data: lineData,
        },
      ],
    });
  });
</script>

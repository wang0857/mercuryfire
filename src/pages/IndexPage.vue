<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn
          color="primary"
          class="q-mt-md"
          @click="addDataHandler"
        >{{ editStart ? '編輯' : '新增' }}</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                v-for="(btn, index) in tableButtons"
                @click="handleClickOption(btn, props.row)"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { QTableProps } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
let blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);

// Get stored data from local storage
const storedData = localStorage.getItem('people')
if (storedData) {
  blockData.value = JSON.parse(storedData)
}

const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

// Control edit status
let editStart = false

// Declare the targeted data
let targetData = ref({
  name: '',
  age: '',
});

// Edit or Add
function addDataHandler(): void {
  if (editStart) {
    targetData.value.name = tempData.value.name
    targetData.value.age = tempData.value.age
    editStart = false

    tempData.value.name = ''
    tempData.value.age = ''

    localStorage.setItem('people', JSON.stringify(blockData.value))
  } else {
    const newData = ref(tempData)
    blockData.value.push({
      name: newData.value.name,
      age: parseInt(newData.value.age)
    })
  
    tempData.value.name = ''
    tempData.value.age = ''

    localStorage.setItem('people', JSON.stringify(blockData.value))
  }
}

// Edit and delete handlers in the table
function handleClickOption(btn: any, data: any) {
  if (btn.status === 'delete') {
    blockData.value = blockData.value.filter(blockData => blockData.name !== data.name)
    
    localStorage.setItem('people', JSON.stringify(blockData.value))
  } else if(btn.status === 'edit') {
    editStart = true
    targetData.value = data
    tempData.value.name = targetData.value.name
    tempData.value.age = targetData.value.age
  }
}

</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>

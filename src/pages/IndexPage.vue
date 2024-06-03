<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-form @reset="onReset">
          <q-input
            lazy-rules
            v-model="tempData.name"
            :rules="[(val) => (val && val.length > 0) || '請輸入姓名']"
            label="姓名"
          />
          <q-input
            lazy-rules
            v-model="tempData.age"
            :rules="[
              (val) => (val !== null && val !== '') || '請輸入年齡',
              (val) => /^[1-9]\d*$/.test(val) || '請輸入正整數',
            ]"
            label="年齡"
          />
          <q-btn
            v-if="!showEdit"
            color="primary"
            @click="addHandler"
            class="q-mt-md"
            >新增</q-btn
          >
          <div
            v-else
            style="width: 150px; display: flex; justify-content: space-between"
          >
            <q-btn color="primary" class="q-mt-md" @click="editHandler"
              >更新</q-btn
            >
            <q-btn color="primary" class="q-mt-md" type="reset" @click="onReset"
              >取消</q-btn
            >
          </div>
        </q-form>
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
        :rows-per-page-options="[0]"
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
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
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
  <q-dialog v-model="alert">
    <q-card style="min-width: 350px">
      <q-card-section>
        <div class="text-h6">提示</div>
      </q-card-section>

      <q-card-section class="q-pt-none">
        是否確定要刪除該筆資料
      </q-card-section>

      <q-card-actions align="right">
        <q-btn
          flat
          label="取消"
          @click="() => (alert = false)"
          color="primary"
        />
        <q-btn flat label="確定" @click="deleteHandler" color="primary" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { ref, onMounted } from 'vue';
const showEdit = ref(false);
const accept = ref(false);
const alert = ref(false);
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const deleteId = ref();
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
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
const onReset = () => {
  tempData.value.name = null;
  tempData.value.age = null;
  showEdit.value = false;
};
const getData = async () => {
  try {
    const data = await axios.get(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );

    blockData.value = data.data;
  } catch (e) {
    console.error(e);
  }
};
const addHandler = async () => {
  try {
    const params = {
      name: tempData.value.name,
      age: tempData.value.age,
    };
    await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', params);
    getData();
    onReset();
  } catch (e) {
    console.error(e);
  }
};
const editHandler = async () => {
  try {
    const params = {
      id: tempData.value.id,
      name: tempData.value.name,
      age: tempData.value.age,
    };
    await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', params);
    getData();
    onReset();
  } catch (e) {
    console.error(e);
  }
};
const deleteHandler = async () => {
  try {
    await axios.delete(
      `https://dahua.metcfire.com.tw/api/CRUDTest/${deleteId.value}`
    );
    getData();
    alert.value = false;
  } catch (e) {
    console.error(e);
  }
};

function handleClickOption(btn, data) {
  // ...
  if (btn.status === 'delete') {
    alert.value = true;
    deleteId.value = data.id;
    console.log('刪除');
  } else if (btn.status === 'edit') {
    showEdit.value = true;
    tempData.value = JSON.parse(JSON.stringify(data));
    console.log('修改');
  }
}
onMounted(() => {
  getData();
});
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

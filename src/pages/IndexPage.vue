<template>
  <q-page class="row q-pt-xl" >
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input ref="nameRef" v-model="tempData.name" label="姓名" :rules="[val => !!val || '姓名不得為空']"/>
        <q-input  ref="ageRef" v-model="tempData.age" label="年齡" :rules="[ val => !!val || '年齡不得為空', val => val > 0 || '年齡必須是正整數']"/>
        <q-btn color="primary" class="q-mt-md"  @click="handleAddOrEdit"> {{ isEditing ? '編輯' : '新增' }}</q-btn>
        <q-btn color="secondary" class="q-mt-md q-ml-md" v-if="isEditing" @click="cancelEdit">取消</q-btn>
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
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { onMounted, ref } from 'vue';
import { Dialog } from 'quasar';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

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
    sortable: true,
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
    sortable: true,
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
const isEditing = ref(false);
const editingId = ref(null);
const nameRef = ref(null);
const ageRef = ref(null);
onMounted(() => {
  toQuery();
});

function toQuery() {
  console.log('查詢');
  axios
    .get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    .then((response) => {
      console.log('response',response);
      if (response.status !== 200) return;
      blockData.value = response.data;
    })
    .catch((error) => {
      console.error(error);
    });
}

function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    isEditing.value = true;
    editingId.value = data.id;
    tempData.value = { ...data };
  }

  if (btn.status === 'delete') {
    Dialog.create({
      title: '提示',
      message: '是否確認刪除該筆資料？',
      ok: {
        label: '確定',
        color: 'primary'
      },
      cancel: {
        label: '取消',
        color: 'secondary'
      },
      persistent: true
    }).onOk(() => {
      // 用戶確認後執行刪除操作
      axios
        .delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`)
        .then(() => {
          blockData.value = blockData.value.filter(item => item.id !== data.id); // 刪除資料
        })
        .catch(error => {
          console.error(error); // 處理錯誤
        });
    }).onCancel(() => {
      console.log('刪除操作已取消');
    });
  }
}

function handleAddOrEdit() {

  const nameValid = nameRef.value?.validate();
  const ageValid = ageRef.value?.validate();

  if (nameValid && ageValid) {
    if (isEditing.value) {
      updateData();
    } else {
      addNewData();
    }
  } else {
    console.log('欄位驗證失敗，請檢查輸入');
  }
}

function addNewData() {
  console.log('新增');
  axios
    .post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
    .then((response) => {
      console.log('response>>>', response);

      if (response.status == 200) {
        toQuery();
        tempData.value = { name: '', age: '' };
      }
    })
    .catch((error) => {
      console.error(error);
    });
}


function updateData() {
  console.log('編輯');
  axios
    .patch('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
    .then((response) => {
      console.log('response>>>', response);

      if (response.status == 200) {
        toQuery();
      }

      tempData.value = { name: '', age: '' };
      isEditing.value = false;
      editingId.value = null;
    })
    .catch((error) => {
      console.error(error);
    });
}

function cancelEdit() {
  tempData.value = { name: '', age: '' };
  isEditing.value = false;
  editingId.value = null;
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

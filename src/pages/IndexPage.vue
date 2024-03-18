<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div v-if="!editVM" class="q-mb-xl">
        <q-input
          v-model="addVM.name"
          label="姓名"
          :rules="[(val) => !!val || '不得空白']"
        />
        <q-input
          type="number"
          v-model.number="addVM.age"
          label="年齡"
          :rules="[
            (val) =>
              (val && Number.isInteger(val)) || '不得空白&限輸入數字(正整數)',
          ]"
        />
        <q-btn color="primary" class="q-mt-md" @click="AddClick"> 新增</q-btn>
      </div>
      <div v-else class="q-mb-xl">
        <q-input
          v-model="editVM.name"
          label="姓名"
          :rules="[(val) => !!val || '不得空白']"
        />
        <q-input
          type="number"
          v-model.number="editVM.age"
          label="年齡"
          :rules="[
            (val) =>
              (val && Number.isInteger(val)) || '不得空白&限輸入數字(正整數)',
          ]"
        />
        <q-btn color="primary" class="q-mt-md" @click="executeEdit">
          更新
        </q-btn>
        <q-btn color="amber" class="q-mt-md" @click="cancelEditClick">
          取消
        </q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="people_list"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template #header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template #body="props">
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
                @click="actionClick(btn, props.row)"
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

    <q-dialog v-model="is_delete_confirm_ModalOpen" persistent>
      <q-card style="width: 500px">
        <q-card-section class="">
          <h4>提示</h4>
          <p class="q-ml-sm">是否確定刪除該筆資料?</p>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn label="取消" v-close-popup />
          <q-btn label="確定" v-close-popup @click="executeDelete" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios'
import { QTableProps } from 'quasar'
import { PersonViewModel } from 'src/models/PersonViewModel'
import { computed, onMounted, ref } from 'vue'

interface btnType {
  label: string
  icon: string
  status: string
}

const is_delete_confirm_ModalOpen = ref(false)
const people_list = ref<PersonViewModel[]>([])
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
])
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
])

const addVM = ref<PersonViewModel>(new PersonViewModel())
const editVM = ref<PersonViewModel>()
const deleteVM = ref<PersonViewModel>()

onMounted(() => {
  getPeopleList()
})

function getPeopleList() {
  axios
    .get<{ result: PersonViewModel[] }>(
      'https://demo.mercuryfire.com.tw:49110/crudTest/a'
    )
    .then((res) => {
      console.log('get people_list', res)
      if (res.status === 200) {
        people_list.value = res.data.result
      }
    })
}

const is_add_valid = computed(() => {
  const { name, age } = addVM.value
  let valid = true
  if (!name || !age?.toString()) {
    valid = false
  }
  return valid
})
function AddClick() {
  if (is_add_valid.value)
    axios
      .post('https://demo.mercuryfire.com.tw:49110/crudTest', addVM.value)
      .then((res) => {
        console.log('add person', res)
        if (res.status === 200) {
          addVM.value = new PersonViewModel()
          getPeopleList()
        }
      })
}

// row Click
function actionClick(btn: btnType, vm: PersonViewModel) {
  console.log('row click', btn, vm)
  if (btn.status === 'delete') {
    deleteClick(vm)
  } else if (btn.status === 'edit') {
    editClick(vm)
  }
}

// 取消編輯
function cancelEditClick() {
  editVM.value = undefined
}
// 點選編輯
function editClick(vm: PersonViewModel) {
  editVM.value = { ...vm }
}
// 執行編輯
function executeEdit() {
  axios
    .patch('https://demo.mercuryfire.com.tw:49110/crudTest', editVM.value)
    .then((res) => {
      console.log('edit person', res)
      if (res.status === 200) {
        editVM.value = undefined
        getPeopleList()
      }
    })
}
function deleteClick(vm: PersonViewModel) {
  is_delete_confirm_ModalOpen.value = true
  deleteVM.value = vm
}

// 執行刪除
function executeDelete() {
  if (deleteVM.value)
    axios
      .delete(
        `https://demo.mercuryfire.com.tw:49110/crudTest/${deleteVM.value.id}`
      )
      .then((res) => {
        console.log('delete person', res)
        if (res.status === 200) {
          getPeopleList()
        }
      })
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

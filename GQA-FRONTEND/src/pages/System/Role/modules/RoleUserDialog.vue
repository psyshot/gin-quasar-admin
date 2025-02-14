<template>
    <q-dialog v-model="roleUserVisible" position="right">
        <q-card style="min-width: 500px; max-width: 45vw; height: 100%;">
            <q-card-section>
                {{ record.role_name }}
            </q-card-section>
            <q-table row-key="id" separator="cell" :rows="tableData" :columns="columns" v-model:pagination="pagination"
                :rows-per-page-options="pageOptions" :loading="loading" @request="onRequest">
                <template v-slot:top="props">
                    <q-btn dense color="primary" @click="showAddUserForm()" :label="$t('Add') + $t('User')" />
                    <q-space />
                    <!-- <q-btn flat round dense :icon="props.inFullscreen ? 'fullscreen_exit' : 'fullscreen'"
                        @click="props.toggleFullscreen" class="q-ml-md" /> -->
                </template>

                <template v-slot:body-cell-actions="props">
                    <q-td :props="props">
                        <div class="q-gutter-xs">
                            <q-btn dense color="negative" @click="handleRemove(props.row)" :label="$t('Delete')" />
                        </div>
                    </q-td>
                </template>
            </q-table>
        </q-card>
        <SelectUserDialog ref="selectUserDialog" @handleSelectUser="handleAddUser" selection="multiple" />
    </q-dialog>
</template>

<script setup>
import useTableData from 'src/composables/useTableData'
import { postAction } from 'src/api/manage'
import { computed, ref } from 'vue'
import SelectUserDialog from 'src/components/GqaSeleteUser/SelectUserDialog'

const url = {
    list: 'role/query-user-by-role',
    removeUser: 'role/remove-role-user',
    addUser: 'role/add-role-user',
}
const columns = computed(() => {
    return [
        { name: 'username', align: 'center', label: t('Username'), field: 'username' },
        { name: 'nickname', align: 'center', label: t('Nickname'), field: 'nickname' },
        { name: 'real_name', align: 'center', label: t('RealName'), field: 'real_name' },
        { name: 'actions', align: 'center', label: t('Actions'), field: 'actions' },
    ]
})
const {
    $q,
    t,
    pagination,
    queryParams,
    pageOptions,
    loading,
    tableData,
    onRequest,
    getTableData,
} = useTableData(url)

const roleUserVisible = ref(false)

const record = ref({})

const show = (row) => {
    pagination.value.sortBy = 'username'
    tableData.value = []
    record.value = row
    queryParams.value.role_code = record.value.role_code
    roleUserVisible.value = true
    getTableData()
}
defineExpose({
    show
})

const selectUserDialog = ref(null)
const showAddUserForm = () => {
    selectUserDialog.value.show(tableData.value)
}
const handleRemove = (row) => {
    $q.dialog({
        title: t('Confirm'),
        message: t('Confirm') + t('Delete') + '?',
        cancel: true,
        persistent: true,
    }).onOk(async () => {
        if (record.value.roleCode === 'super-admin' && row.username === 'admin') {
            $q.notify({
                type: 'negative',
                message: t('CanNotDeleteThis'),
            })
            return false
        }
        const res = await postAction(url.removeUser, {
            role_code: record.value.role_code,
            username: row.username,
        })
        if (res.code === 1) {
            $q.notify({
                type: 'positive',
                message: res.message,
            })
        }
        getTableData()
    })
}
const handleAddUser = (event) => {
    const usernameList = []
    for (let i of event) {
        usernameList.push(i.username)
    }
    postAction(url.addUser, {
        role_code: record.value.role_code,
        username: usernameList,
    }).then((res) => {
        if (res.code === 1) {
            $q.notify({
                type: 'positive',
                message: res.message,
            })
        }
        getTableData()
    })
}
</script>

<template>
    <div style="padding:10px">
        <q-card inline class="fit shadow-6">
            <q-card-main style="height:80%">
                <q-table ref="table" color="primary" :data="serverData" :columns="columns" selection="multiple" :selected.sync="selected" row-key="id" :pagination.sync="serverPagination" @request="request" :loading="loading" rows-per-page-label="每页数据" no-data-label="没有数据">
                    <template slot="top-left" slot-scope="props">
                        <q-input v-model="filter.name"  type="text" prefix="用户名称：" />&nbsp;&nbsp;
                        <q-input v-model="filter.email"  type="text" prefix="用户邮箱：" />&nbsp;&nbsp;
                        <q-btn push dense color="primary" icon="search" @click="search">查询</q-btn>&nbsp;&nbsp;
                        <q-btn push dense color="primary" icon="add" @click="addUser">新增</q-btn>
                    </template>
                    <template slot="top-right" slot-scope="props">
                        <q-btn flat round dense :icon="props.inFullscreen ? 'fullscreen_exit' : 'fullscreen'" @click="props.toggleFullscreen" />
                    </template>
                    <template slot="top-selection" slot-scope="props">
                        <q-btn color="negative" flat round delete icon="delete" @click="delUsers" />
                    </template>
                    <q-td slot="body-cell-id" slot-scope="props" :props="props" style="width:100px">
                        <q-btn small round push glossy dense icon="edit" color="primary" @click="editUser(props.value)"></q-btn>
                        <q-btn small round push glossy dense icon="delete" color="red" @click="delUser(props.value)"></q-btn>
                    </q-td>
                </q-table>
            </q-card-main>
        </q-card>
        <q-modal v-model="editModal" :content-css="{minWidth: '400px', minHeight: '500px'}">
            <q-modal-layout>
                <q-toolbar slot="header">
                    <q-btn flat round dense @click="editModal = false" icon="reply" />
                    <q-toolbar-title>
                        编辑
                    </q-toolbar-title>
                </q-toolbar>
                <q-toolbar slot="footer">
                    <q-toolbar-title>
                    </q-toolbar-title>
                    <q-btn round @click="saveUser">保存</q-btn>
                    <q-btn round color="red" @click="editModal = false">取消</q-btn>
                </q-toolbar>

                <div class="layout-padding">
                    <q-field :count="15">
                        <q-input v-model="tempUser.name" float-label="账号名称" />
                    </q-field>
                    <q-field :count="15">
                        <q-input v-model="tempUser.trueName" float-label="用户名称" />
                    </q-field>
                    <q-field :count="15">
                        <q-input v-model="tempUser.email" float-label="用户邮箱" />
                    </q-field>
                     <q-field :count="15">
                        <q-input v-model="tempUser.phone" float-label="Phone" />
                    </q-field>
                </div>
            </q-modal-layout>
        </q-modal>
    </div>
</template>

<script>
import {
  getUserPagedList,
  delUser,
  delUsers,
  saveUser
} from "@/service/user/user";
export default {
  name: "user",
  data() {
    return {
      serverData: [],
      serverPagination: {
        page: 1,
        rowsNumber: 0, // specifying this determines pagination is server-side
        rowsPerPage: 10 // current rows per page being displayed
      },
      columns: [
        {
          name: "name",
          required: true,
          label: "账号名称",
          align: "left",
          field: "name",
          sortable: true
        },
        {
          name: "trueName",
          label: "用户名称",
          field: "trueName",
          sortable: true,
          align: "left"
        },
        {
          name: "email",
          label: "邮箱",
          field: "email",
          sortable: true,
          align: "left"
        },
        {
          name: "phone",
          label: "Phone",
          field: "phone",
          sortable: true,
          align: "left"
        },
        {
          name: "id",
          required: true,
          label: "操作",
          align: "left",
          field: "id"
        }
      ],
      filter: {
        name: "",
        email: ""
      },
      selected: [],
      loading: false,
      editModal: false,
      tempUser: {
        id: 0,
        name: "",
        trueName: "",
        email: "",
        phone:""
      }
    };
  },
  methods: {
    async request(props) {
      this.loading = true;
      this.serverPagination = props.pagination;
      let table = this.$refs.table,
        { page, rowsPerPage, sortBy, descending } = props.pagination;
      let query = {
        pageIndex: page,
        pageSize: rowsPerPage,
        sortBy: sortBy,
        descending: descending,
        filter: this.filter
      };
      let dataRes = await getUserPagedList(query);
      let data = dataRes.data.data;
      this.serverPagination.rowsNumber = data.totalCount;
      this.serverData = data.rows;
      setTimeout(() => {
        this.loading = false;
      }, 500);
    },
    async delUser(id) {
      try {
        await this.$q.dialog({
          title: "删除",
          message: "确认执行删除操作？",
          position: "right",
          ok: {
            push: true,
            label: "删除"
          },
          cancel: {
            push: true,
            color: "negative",
            label: "取消"
          }
        });
        await delUser({ id: id });
        this.$q.notify({
          type: "positive",
          message: "删除成功",
          position: "bottom-right"
        });
        this.search();
      } catch (e) {}
    },
    async delUsers() {
      try {
        await this.$q.dialog({
          title: "批量删除",
          message: "确认执行批量删除操作？",
          ok: {
            push: true,
            label: "删除"
          },
          cancel: {
            push: true,
            color: "negative",
            label: "取消"
          }
        });
        await delUsers({
          ids: JSON.stringify(
            this.selected.map(s => {
              return s.id;
            })
          )
        });
        this.$q.notify({
          type: "positive",
          message: "批量删除成功",
          position: "bottom-right"
        });
        this.selected = [];
        this.search();
      } catch (e) {}
    },
    editUser(id) {
      let editRole = this.serverData.filter(s => {
        return s.id === id;
      });
      if (editRole && editRole.length > 0) {
        this.tempUser = JSON.parse(JSON.stringify(editRole[0]));
      } else {
        return true;
      }
      this.editModal = true;
    },
    async saveUser() {
      await saveUser(this.tempUser);
      this.$q.notify({
        type: "positive",
        message: "保存成功",
        position: "bottom-right"
      });
      this.editModal = false;
      this.search();
    },
    search() {
      this.request({
        pagination: this.serverPagination,
        filter: this.filter
      });
    },
    addUser() {
      this.tempUser = {};
      this.editModal = true;
    }
  },
  mounted() {
    this.request({
      pagination: this.serverPagination,
      filter: this.filter
    });
  }
};
</script>
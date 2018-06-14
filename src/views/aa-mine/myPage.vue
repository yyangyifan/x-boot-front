<style lang="less">
@import "./myPage.less";
</style>
<template>
    <div class="search">
      <Row>
          <Col>
              <Card>
                  <Row>
                      <Form ref="searchForm" :model="searchForm" inline :label-width="70" class="search-form">
                        <Form-item label="类型" prop="type">
                          <Select v-model="searchForm.type" placeholder="???" clearable style="width: 150px">
                            <Option value="0">000</Option>
                            <Option value="1">001</Option>
                          </Select>
                        </Form-item>

                        <Form-item label="有效性" prop="status">
                          <Select v-model="searchForm.status" placeholder="???" clearable style="width: 150px">
                            <Option value="0">有效</Option>
                            <Option value="1">???</Option>
                          </Select>
                        </Form-item>

                        <Form-item label="时间" prop="status">
                          <DatePicker type="daterange" format="yyyy/MM/dd" clearable @on-change="selectDateRange" placeholder="????" style="width: 200px"></DatePicker>
                        </Form-item>

                        <Form-item label="搜索">
                          <Input type="text" v-model="searchKey" clearable placeholder="请输入搜索关键词" style="width: 300px"/>
                        </Form-item>


                          <Form-item style="margin-left:-35px;">
                            <Button @click="handleSearch" type="primary" icon="search">搜索</Button>
                            <Button @click="handleReset" type="ghost" >重置</Button>
                            <a class="drop-down" @click="dropDown">{{dropDownContent}}
                              <Icon :type="dropDownIcon"></Icon>
                            </a>
                          </Form-item>
                      </Form>
                  </Row>
                  <Row class="operation">
                      <Button @click="addRow" type="primary" icon="plus-round">新增</Button>
                    <Button @click="EditRow" type="ghost" icon="trash-a">修改</Button>
                    <Button @click="delRow" type="ghost" icon="trash-a">删除</Button>
                  </Row>

                  <Row class="margin-top-10 searchable-table-con1">
                      <Table :loading="loading" border :columns="columns" :data="data" sortable="custom" @on-sort-change="changeSort" @on-selection-change="showSelect" ref="table"></Table>
                      <Table :columns="columns" :data="exportData" ref="exportTable" style="display:none"></Table>
                  </Row>
                  <Row type="flex" justify="end" class="code-row-bg page">
                      <Page :current="this.searchForm.pageNumber" :total="total" :page-size="this.searchForm.pageSize" @on-change="changePage" @on-page-size-change="changePageSize" :page-size-opts="[10,20,50,100]" size="small" show-total show-elevator show-sizer></Page>
                  </Row>
              </Card>
          </Col>
      </Row>

      <Modal :title="modalTitle" v-model="userModalVisible" :mask-closable='false' :width="500">
            <Form ref="userForm" :model="userForm" :label-width="70" :rules="userFormValidate">
                <FormItem label="用户名" prop="username">
                    <Input v-model="userForm.username"/>
                </FormItem>
                <FormItem label="密码" prop="password" v-if="modalType===0" :error="errorPass">
                    <Input type="password" v-model="userForm.password"/>
                </FormItem>
                <FormItem label="邮箱" prop="email">
                    <Input v-model="userForm.email"/>
                </FormItem>
                <FormItem label="手机号" prop="mobile">
                    <Input v-model="userForm.mobile"/>
                </FormItem>
                <FormItem label="性别" prop="sex">
                  <RadioGroup v-model="userForm.sex">
                    <Radio :label="1">男</Radio>
                    <Radio :label="0">女</Radio>
                  </RadioGroup>
                </FormItem>
                <FormItem label="用户类型" prop="type">
                  <Select v-model="userForm.type" placeholder="请选择">
                    <Option :value="0">普通用户</Option>
                    <Option :value="1">管理员</Option>
                  </Select>
                </FormItem>
                <FormItem label="角色分配" prop="roles">
                  <Select v-model="userForm.roles" multiple @on-change="selectRoles">
                      <Option v-for="item in roleList" :value="item.id" :key="item.id">{{ item.name }}</Option>
                  </Select>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="text" @click="cancelUser">取消</Button>
                <Button type="primary" :loading="submitLoading" @click="submitUser">提交</Button>
            </div>
        </Modal>
    </div>
</template>

<script>
export default {
  name: "user-manage",
  data() {

    // const validateMobile = (rule, value, callback) => {
    //   var reg = /^[1][3,4,5,7,8][0-9]{9}$/;
    //   if (!reg.test(value)) {
    //     callback(new Error("手机号格式错误"));
    //   } else {
    //     callback();
    //   }
    // };
    return {
      loading: true,
      drop: false,
      dropDownContent: "展开",
      dropDownIcon: "chevron-down",
      selectCount: 0,
      selectList: [],
      searchForm: {
        username: "",
        mobile: "",
        email: "",
        sex: "",
        type: "",
        status: "",
        pageNumber: 1,
        pageSize: 10,
        sort: "createTime",
        order: "desc",
        startDate: "",
        endDate: "",
        searchBar:""
      },



      sdkForm: {
        id: "",
        sdkName: "",
        type: "",
        hostname:"",
        source:"",
        availability:"",
        date:"",
        intro:"",
        list:[]

      },

      modalType: 0,
      userModalVisible: false,
      modalTitle: "",
      userForm: {
        sex: 1,
        roles: []
      },
      userRoles: [],
      roleList: [],
      errorPass: "",

      sdkFormValidate: {},

      submitLoading: false,

      columns: [
        {
          type: "selection",
          width: 50,
          align: "center"
        },
        {
          title: "序号",
          key: "id",
          width: 60,
          sortable: true
        },
        {
          title: "SDK名称",
          key: "sdkName",
          width: 100,
          sortable: true
        },
        {
          title: "类型",
          key: "email",
          width: 100,
          sortable: false
        },
        {
          title: "域名",
          key: "hostname",
          width: 150,
          sortable: false
        },
        {
          title: "来源",
          key: "source",
          width: 150,
          sortable: false
        },
        {
          title: "有效性",
          key: "availability",
          width: 50,
          sortable: false
        },
        {
          title: "入库时间",
          key: "date",
          width: 100,
          sortable: false
        },
        {
          title: "简介",
          key: "source",
          width: 100,
          sortable: false
        },
        {
          title: "调用名单",
          key: "action",
          width: 100,
          align: "center",
          render: (h, params) => {
            if (params.row.status === 0) {
              return h("div", [
                h(
                  "Button",
                  {
                    props: {
                      type: "primary",
                      size: "small"
                    },
                    style: {
                      marginRight: "5px"
                    },
                    on: {
                      click: () => {
                        this.edit(params.row);
                      }
                    }
                  },
                  "图标"
                )
              ]);
            } else {
              return h("div", [
                h(
                  "Button",
                  {
                    props: {
                      type: "primary",
                      size: "small"
                    },
                    style: {
                      marginRight: "5px"
                    },
                    on: {
                      click: () => {
                        this.edit(params.row);
                      }
                    }
                  },
                  "图标"
                )
              ]);
            }
          }
        }
      ],
      data: [],
      exportData: [],
      total: 0
    };
  },
  methods: {
    init() {
      this.getUserList();
    },
    changePage(v) {
      this.searchForm.pageNumber = v;
      this.getUserList();
    },
    changePageSize(v) {
      this.searchForm.pageSize = v;
      this.getUserList();
    },
    selectDateRange(v) {
      if (v) {
        this.searchForm.startDate = v[0];
        this.searchForm.endDate = v[1];
      }
    },
    getUserList() {
      // 多条件搜索用户列表
      this.loading = true;
      this.getRequest("/user/getByCondition", this.searchForm).then(res => {
        this.loading = false;
        if (res.success === true) {
          this.data = res.result.content;
          this.total = res.result.totalElements;
        }
      });
    },
    handleSearch() {
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      this.init();
    },
    handleReset() {
      this.$refs.searchForm.resetFields();
      this.searchForm.pageNumber = 1;
      this.searchForm.pageSize = 10;
      // 重新加载数据
      this.init();
    },
    changeSort(e) {
      this.searchForm.sort = e.key;
      this.searchForm.order = e.order;
      if (e.order === "normal") {
        this.searchForm.order = "";
      }
      this.init();
    },
    getRoleList() {
      this.getRequest("/role/getAllList").then(res => {
        if (res.success === true) {
          this.roleList = res.result;
        }
      });
    },
    handleDropdown(name) {
      if (name === "exportData") {
        if (this.selectCount <= 0) {
          this.$Message.warning("您还未选择要导出的数据");
          return;
        }
        this.$Modal.confirm({
          title: "确认导出",
          content: "您确认要导出所选 " + this.selectCount + " 条数据?",
          onOk: () => {
            this.$refs.exportTable.exportCsv({
              filename: "用户数据"
            });
          }
        });
      } else if (name === "refresh") {
        this.getUserList();
      }
    },
    selectRoles(v) {},
    cancelUser() {
      this.userModalVisible = false;
    },
    submitUser() {
      this.$refs.userForm.validate(valid => {
        if (valid) {
          let url = "/user/admin/add";
          if (this.modalType === 1) {
            // 编辑用户
            url = "/user/admin/edit";
          }
          if (this.modalType === 0) {
            if (this.userForm.password.length < 6) {
              this.errorPass = "密码不能为空且长度不得少于6位";
              return;
            }
          }
          this.submitLoading = true;
          this.postRequest(url, this.userForm).then(res => {
            this.submitLoading = false;
            if (res.success === true) {
              this.$Message.success("操作成功");
              this.init();
              this.userModalVisible = false;
            }
          });
        }
      });
    },
    addRow() {
      this.modalType = 0;
      this.modalTitle = "添加用户";
      this.$refs.userForm.resetFields();
      this.userModalVisible = true;
    },
    edit(v) {
      this.modalType = 1;
      this.modalTitle = "编辑用户";
      this.$refs.userForm.resetFields();
      // 转换null为""
      for (let attr in v) {
        if (v[attr] === null) {
          v[attr] = "";
        }
      }
      let str = JSON.stringify(v);
      let userInfo = JSON.parse(str);
      this.userForm = userInfo;
      let selectRolesId = [];
      this.userForm.roles.forEach(function(e) {
        selectRolesId.push(e.id);
      });
      this.userForm.roles = selectRolesId;
      this.userModalVisible = true;
    },
    enable(v) {
      this.$Modal.confirm({
        title: "确认启用",
        content: "您确认要启用用户 " + v.username + " ?",
        onOk: () => {
          this.postRequest("/user/admin/enable/" + v.id).then(res => {
            if (res.success === true) {
              this.$Message.success("操作成功");
              this.init();
            }
          });
        }
      });
    },
    disable(v) {
      this.$Modal.confirm({
        title: "确认禁用",
        content: "您确认要禁用用户 " + v.username + " ?",
        onOk: () => {
          this.postRequest("/user/admin/disable/" + v.id).then(res => {
            if (res.success === true) {
              this.$Message.success("操作成功");
              this.init();
            }
          });
        }
      });
    },
    remove(v) {
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除用户 " + v.username + " ?",
        onOk: () => {
          this.deleteRequest("/user/delByIds", { ids: v.id }).then(res => {
            if (res.success === true) {
              this.$Message.success("删除成功");
              this.init();
            }
          });
        }
      });
    },
    dropDown() {
      if (this.drop) {
        this.dropDownContent = "展开";
        this.dropDownIcon = "chevron-down";
      } else {
        this.dropDownContent = "收起";
        this.dropDownIcon = "chevron-up";
      }
      this.drop = !this.drop;
    },
    showSelect(e) {
      this.exportData = e;
      this.selectList = e;
      this.selectCount = e.length;
    },
    clearSelectAll() {
      this.$refs.table.selectAll(false);
    },
    delRow() {
      if (this.selectCount <= 0) {
        this.$Message.warning("您还未选择要删除的数据");
        return;
      }
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除所选的 " + this.selectCount + " 条数据?",
        onOk: () => {
          let ids = "";
          this.selectList.forEach(function(e) {
            ids += e.id + ",";
          });
          ids = ids.substring(0, ids.length - 1);
          this.deleteRequest("/user/delByIds", { ids: ids }).then(res => {
            if (res.success === true) {
              this.$Message.success("删除成功");
              this.init();
            }
          });
        }
      });
    }
  },
  mounted() {
    this.init();
    this.getRoleList();
  }
};
</script>

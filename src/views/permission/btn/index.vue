<template>
  <div class="app-container">
    <el-row>
      <el-form :inline="true" :model="selectInfo">
        <el-form-item label="按钮名">
          <el-input v-model="selectInfo.btnName" placeholder="按钮名"/>
        </el-form-item>
        <el-form-item label="按钮标识">
          <el-input v-model="selectInfo.btnCode" placeholder="按钮标识"/>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getBtnList">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="success" @click="addBtnInfo(null)">新增按钮</el-button>
        </el-form-item>
      </el-form>
    </el-row>
    <el-row>
      <el-table v-loading="listLoading" :data="btnList" border fit highlight-current-row style="width: 100%">
        <el-table-column align="center" label="ID" width="80">
          <template slot-scope="scope">
            <span>{{ scope.row.id }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="按钮名">
          <template slot-scope="scope">
            <span>{{ scope.row.btnName }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="按钮标识">
          <template slot-scope="scope">
            <span>{{ scope.row.btnCode }}</span>
          </template>
        </el-table-column>


        <el-table-column align="center" label="创建人">
          <template slot-scope="scope">
            <span>{{ scope.row.createUserName }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="创建时间" width="180">
          <template slot-scope="scope">
            <span>{{ !scope.row.createTime || scope.row.createTime==0 ? '': parseTime(scope.row.createTime) }}</span>
          </template>
        </el-table-column>

<!--        <el-table-column align="center" label="操作" width="440">-->
<!--          <template slot-scope="scope">-->
<!--            <el-button-->
<!--              type="primary"-->
<!--              size="small"-->
<!--              icon="el-icon-edit"-->
<!--              @click="addBtnInfo(scope.row)"-->
<!--            >-->
<!--              修改-->
<!--            </el-button>-->

<!--            <el-button-->
<!--              type="danger"-->
<!--              size="small"-->
<!--              icon="el-icon-delete"-->
<!--              @click="deleteBtn(scope.row)"-->
<!--            >-->
<!--              删除-->
<!--            </el-button>-->
<!--          </template>-->
<!--        </el-table-column>-->
      </el-table>
    </el-row>
    <el-row>
      <el-pagination
        :total="selectInfo.total"
        :current-page="selectInfo.currentPage"
        :page-size="selectInfo.pageSize"
        align="center"
        background
        layout="prev, pager, next"
        @current-change="pageChange"
      />
    </el-row>
    <el-dialog v-el-drag-dialog :visible.sync="dialogTableVisible" title="按钮信息" @dragDialog="handleDrag">
      <el-form ref="btnForm" :inline="false" :model="btnInfo" :rules="btnRules" label-width="120px">

        <el-form-item label="按钮编码" prop="btnCode" v-if="showCode">
          <el-input ref="btnCode" v-model="btnInfo.btnCode" placeholder="按钮编码,设置后不能修改"/>
        </el-form-item>

        <el-form-item label="按钮名" prop="btnName">
          <el-input ref="name" v-model="btnInfo.btnName" placeholder="按钮名"/>
        </el-form-item>

        <el-form-item label="按钮标识" prop="btnCode">
          <el-input ref="name" v-model="btnInfo.btnCode" placeholder="按钮标识，作为权限控制"/>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="saveBtn">保存</el-button>
        </el-form-item>

      </el-form>
    </el-dialog>
  </div>
</template>

<script>
    import elDragDialog from '@/directive/el-drag-dialog'
    import {
        getAllBtn, addBtn, deleteBtn, btnList
    } from '@/api/btn'
    import {parseTime} from '@/utils'
    import constant from './constant'
    import commonUtils from '@/utils/commonUtils'
    import {getAllMenu} from "@/api/menu";

    export default {
        name: 'Btn',
        directives: {elDragDialog},
        data() {
            let data = {
                btnRules: {
                    btnCode: [{required: true, message: '请输入按钮编码', trigger: 'blur'}],
                    btnName: [{required: true, message: '请输入按钮名', trigger: 'blur'}]
                },
                btnList: [],
                selectInfo: {
                    currentPage: 1,
                    pageSize: 10,
                    total: 0,
                    btnName: null,
                    btnCode: null
                },
                dialogTableVisible: false,
                btnInfo: {
                    btnName: null,
                    btnCode: null
                },
                listLoading: false,
                menuList: [],
                showCode: true,
            }
            return data
        },
        mounted() {
            this.getBtnList()
        },
        methods: {
            pageChange(currentPage) {
                this.selectInfo.currentPage = currentPage
                this.getBtnList()
            },
            parseTime: parseTime,
            getBtnList() {
                btnList(this.selectInfo).then(response => {
                    this.selectInfo.currentPage = response.pageInfo.currentPage
                    this.selectInfo.pageSize = response.pageInfo.pageSize
                    this.selectInfo.total = response.pageInfo.total
                    this.btnList = response.btnList
                })
            },
            // v-el-drag-dialog onDrag callback function
            handleDrag() {
                this.$refs.select.blur()
            },
            addBtnInfo(row) {
                commonUtils.clearObject(this.btnInfo)
                this.menuList.splice(0)
                getAllMenu().then((response) => {
                    this.menuList = response
                    if (row) {
                        this.showCode = false
                        this.btnCode = null
                        this.btnInfo.id = row.id
                        this.btnInfo.btnName = row.btnName
                        this.btnInfo.btnCode = row.btnCode
                    }
                    this.dialogTableVisible = true
                })
            },
            saveBtn() {
                this.$refs.btnForm.validate(valid => {
                    if (valid) {
                        addBtn(this.btnInfo).then(() => {
                            this.$message({
                                message: '保存成功',
                                type: 'success'
                            });
                            this.getBtnList()
                            this.dialogTableVisible = false
                        })
                    } else {
                        this.$message.error('表单填写错误')
                        return false
                    }
                })
            },
            deleteBtn(row) {
                this.$confirm('此操作将永久删除, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    deleteBtn({btnId: row.id}).then(() => {
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getBtnList()
                    })
                })
            }
        }
    }
</script>

<style lang="scss" scoped>

</style>

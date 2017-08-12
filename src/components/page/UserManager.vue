<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-menu"></i> 表格</el-breadcrumb-item>
                <el-breadcrumb-item>基础表格</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="handle-box">
            <el-button type="primary" icon="plus" class="handle-del mr10" @click="showAddDialog">添加</el-button>
            <el-select v-model="select_cate" placeholder="筛选省份" class="handle-select mr10">
                <el-option key="1" label="广东省" value="广东省"></el-option>
                <el-option key="2" label="湖南省" value="湖南省"></el-option>
            </el-select>
            <el-input v-model="select_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>
            <el-button type="primary" icon="search" @click="search">搜索</el-button>
        </div>
        <el-table :data="data" border style="width: 100%" ref="multipleTable" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55"></el-table-column>
            <el-table-column prop="createTime" label="创建日期" sortable width="150" :formatter="formatter">
            </el-table-column>
            <el-table-column prop="username" label="姓名" width="120">
            </el-table-column>
            <el-table-column prop="password" label="密码">
            </el-table-column>
            <el-table-column prop="email" label="邮箱">
            </el-table-column>
            <el-table-column label="操作" width="180">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button size="small" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <div class="pagination">
            <el-pagination @current-change="handleCurrentChange" layout="prev, pager, next" :total="total">
            </el-pagination>
        </div>
    
        <el-dialog title="提示" :visible.sync="addDailogVisible" size="small" :before-close="handleClose">
            <el-form ref="form" :model="oneData" label-width="80px">
                <el-form-item label="用户名">
                    <el-input v-model="oneData.username"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input v-model="oneData.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱">
                    <el-input v-model="oneData.email"></el-input>
                </el-form-item>

            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDailogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addOrChange">确 定</el-button>
            </span>
        </el-dialog>
    
    </div>
</template>

<script>
import dateformat from "dateformat";
export default {
    data() {
        return {
            url: '/api/users',
            tableData: [],
            cur_page: 1,
            multipleSelection: [],
            select_cate: '',
            select_word: '',
            del_list: [],
            is_search: false,
            total: 10,
            addDailogVisible: false,
            oneData: {}
        }
    },
    created() {
        this.getData();
    },
    computed: {
        data() {
            const self = this;
            return self.tableData.filter(function (d) {
                let is_del = false;
                for (let i = 0; i < self.del_list.length; i++) {
                    if (d.username === self.del_list[i].username) {
                        is_del = true;
                        break;
                    }
                }
                if (!is_del) {
                    if (d.password.indexOf(self.select_cate) > -1 &&
                        (d.username.indexOf(self.select_word) > -1 ||
                            d.password.indexOf(self.select_word) > -1)
                    ) {
                        return d;
                    }
                }
            })
        }
    },
    methods: {
        addOrChange() {
            let self = this;
            if (process.env.NODE_ENV === 'development') {
                self.url = '/api/users';
            };
            if(self.oneData._id){
                self.$axios.put(self.url+'/'+self.oneData._id, self.oneData ).then((res) => {
                    self.$message.info('更新成功！');
                    self.addDailogVisible = false;
                    self.getData();
                });
            }else{
                self.$axios.post(self.url, self.oneData).then((res) => {
                    self.addDailogVisible = false;
                    self.getData();
                });
            }
            
        },
        showAddDialog() {
            this.oneData = {};
            this.addDailogVisible = true;
        },
        handleCurrentChange(val) {
            this.cur_page = val;
            this.getData();
        },
        getData() {
            let self = this;
            if (process.env.NODE_ENV === 'development') {
                self.url = '/api/users';
            };
            self.$axios.get(self.url, { params: { page: self.cur_page } }).then((res) => {
                self.tableData = res.data.list;
                self.total = res.data.total;
            })
        },
        search() {
            this.is_search = true;
        },
        formatter(row, column) {
            return dateformat(row.createTime, 'yyyy-mm-dd');
        },
        filterTag(value, row) {
            return row.tag === value;
        },
        handleEdit(index, row) {
            //this.$message('编辑第' + (index + 1) + '行');
            this.oneData = row;
            this.addDailogVisible = true;
        },
        handleDelete(index, row) {
            //this.$message.error('删除第' + (index + 1) + '行');
            let self = this;
            self.$axios.delete(self.url + '/' + row._id).then((res) => {
                self.$message.info('删除成功！');
                self.getData();
            });
        },
        handleClose() {
            //alert('关闭');
            this.addDailogVisible = false;
        },
        handleSelectionChange(val) {
            this.multipleSelection = val;
        }
    }
}
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
</style>
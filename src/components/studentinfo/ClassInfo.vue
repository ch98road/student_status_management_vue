<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item> <i class="el-icon-lx-cascades"></i> 学生成绩表 </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-input v-model="query.sid" placeholder="学号" class="handle-input mr10"></el-input>
                <el-input v-model="query.studentName" placeholder="姓名" class="handle-input mr10"></el-input>
                <el-input v-model="query.cname" placeholder="课程名" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                :default-sort = "{prop: 'cname', order: 'descending'}"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center" sortable></el-table-column>
                <el-table-column prop="sid" label="学号" width="55" align="center" sortable></el-table-column>
                <el-table-column prop="studentName" label="姓名" sortable></el-table-column>
                
                <el-table-column prop="cname" label="课程名称" sortable></el-table-column>
                <el-table-column prop="teacherName" label="授课老师名称" sortable></el-table-column>
                <el-table-column label="分数" sortable>
                    <template slot-scope="scope" >{{ scope.row.grade }}</template>
                </el-table-column>
                <el-table-column prop="credit" label="学分" sortable></el-table-column>
                <el-table-column prop="rebuild" label="重修标记" sortable></el-table-column>

                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">录入分数</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination
                    background
                    layout="total, prev, pager, next"
                    :current-page="query.pageIndex"
                    :page-size="query.pageSize"
                    :total="pageTotal"
                    @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="录入分数" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="学号">
                    <el-input v-model="form.sid" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="学生姓名">
                    <el-input v-model="form.studentName" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="课程名称">
                    <el-input v-model="form.cname" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="授课老师">
                    <el-input v-model="form.teacherName" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="分数">
                    <el-input v-model="form.grade"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { fetch_grade, update_grade ,add_student} from '../../api/index';
import Axios from 'axios';
export default {
    name: 'basetable',
    data() {
        return {
            query: {
                sid: '',
                studentName: '',
                cname: '',
                pageIndex: 1,
                pageSize: 10
            },
            options_class: [
                {
                    value: '1',
                    label: '1'
                },
                {
                    value: '2',
                    label: '2'
                },
                {
                    value: '3',
                    label: '3'
                }
            ],
            options_major: [
                {
                    value: '计算机科学与技术',
                    label: '计算机科学与技术'
                },
                {
                    value: '数字媒体',
                    label: '数字媒体'
                }
            ],
            value_class: '',
            value_major: '',
            tableData: [],
            totalData:[],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            addVisible: false,
            pageTotal: 0,
            form: {},
            form_add:{},
            origin: {},
            idx: -1,
            id: -1
        };
    },
    created() {
        this.getData();
    },
    methods: {
        // 获取 easy-mock 的模拟数据
        getData() {
            fetch_grade(this.query).then(res => {
                // console.log(this.query);
                // console.log('this.res===:');
                // console.log(res);
                this.tableData = res.data.data;
                // for(let i =(this.query.pageIndex-1)*this.query.pageSize+1;i<=this.query.pageIndex*this.query.pageSize;i++){
                //     this.tableData[i-(this.query.pageIndex-1)*this.query.pageSize-1] = this.totalData[i];
                // }
                // console.log(this.tableData);
                this.pageTotal = res.data.pagetotal || 10;
            });

        },
        // 触发搜索按钮
        handleSearch() {
            this.$set(this.query, 'pageIndex', 1);
            this.getData();
        },

        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.origin['sid'] = row['sid'];
            this.origin['studentName'] = row['studentName'];
            this.origin['cname'] = row['cname'];
            this.origin['teacherName'] = row['teacherName'];
            this.origin['grade'] = row['grade'];
            this.editVisible = true;
        },
        // 保存编辑
        saveEdit() {
            this.editVisible = false;
            this.$message.success(`修改第 ${this.idx + 1} 行成功`);

            this.$set(this.tableData, this.idx, this.form);
            // console.log(this.form);
            // console.log("this.row:",this.row);
            // console.log("this.origin:",this.origin);
            let data = {};
            data['sid'] = this.form['sid'];
            data['cname']= this.form['cname'];
            data['grade'] = this.form['grade']
            data['pageIndex'] = this.query.pageIndex
            data['pageSize'] = this.query.pageSize
            console.log(data)
            if (this.form['grade'] != this.origin['grade']) {
                update_grade(data).then(res=>{
                    console.log("sssssssssss",res);
                    this.tableData = res.data.data;
                    this.pageTotal = res.data.pagetotal || 10;
                })
            }
        
        },
        // 分页导航
        handlePageChange(val) {
            this.$set(this.query, 'pageIndex', val);
            this.getData();
        }
    }
};
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
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>

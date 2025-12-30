<template>
    <div class="mod-biz-feedback">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-select v-model="dataForm.feedbackType" placeholder="反馈类型" clearable style="width: 150px;">
                    <el-option label="产品" value="PRODUCT"></el-option>
                    <el-option label="人物形象" value="CHARACTER"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-select v-model="dataForm.status" placeholder="处理状态" clearable style="width: 120px;">
                    <el-option label="未处理" :value="0"></el-option>
                    <el-option label="已处理" :value="1"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-input v-model="dataForm.feedbackDesc" placeholder="反馈描述" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
            </el-form-item>
        </el-form>
        <el-table :data="dataList" border v-loading="dataListLoading" :row-class-name="tableRowClassName" style="width: 100%;">
            <el-table-column prop="feedbackId" header-align="center" align="center" label="反馈ID" width="150">
            </el-table-column>
            <el-table-column prop="contentId" header-align="center" align="center" label="内容ID" width="150">
            </el-table-column>
            <el-table-column prop="feedbackType" header-align="center" align="center" label="反馈类型" width="120" :formatter="feedbackTypeFormat">
            </el-table-column>
            <el-table-column prop="feedbackDesc" header-align="center" align="center" label="反馈描述" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column prop="userOpenid" header-align="center" align="center" label="用户OpenID" width="200" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column prop="status" header-align="center" align="center" label="处理状态" width="100" :formatter="statusFormat">
            </el-table-column>
            <el-table-column prop="handleRemark" header-align="center" align="center" label="处理备注" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column prop="createTime" header-align="center" align="center" label="创建时间" width="160">
            </el-table-column>
            <el-table-column prop="updateTime" header-align="center" align="center" label="更新时间" width="160">
            </el-table-column>
            <el-table-column fixed="right" header-align="center" align="center" width="120" label="操作">
                <template slot-scope="scope">
                    <el-button v-if="scope.row.status === 0" type="text" size="small" @click="handleFeedback(scope.row)">处理</el-button>
                    <span v-else>-</span>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination
            @size-change="sizeChangeHandle"
            @current-change="currentChangeHandle"
            :current-page="pageIndex"
            :page-sizes="[10, 20, 50, 100]"
            :page-size="pageSize"
            :total="totalPage"
            layout="total, sizes, prev, pager, next, jumper">
        </el-pagination>
        <!-- 处理反馈弹窗 -->
        <el-dialog title="处理反馈" :close-on-click-modal="false" :visible.sync="handleVisible" width="600px">
            <el-form :model="handleForm" :rules="handleRule" ref="handleForm" label-width="100px">
                <el-form-item label="反馈ID">
                    <el-input v-model="handleForm.feedbackId" disabled></el-input>
                </el-form-item>
                <el-form-item label="反馈类型">
                    <el-input v-model="handleForm.feedbackTypeText" disabled></el-input>
                </el-form-item>
                <el-form-item label="反馈描述">
                    <el-input v-model="handleForm.feedbackDesc" type="textarea" :rows="3" disabled></el-input>
                </el-form-item>
                <el-form-item label="处理备注" prop="handleRemark">
                    <el-input v-model="handleForm.handleRemark" type="textarea" :rows="4" placeholder="请输入处理备注"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer" style="display: flex; justify-content: space-between;">
                <el-button type="primary" @click="regenerateHandle()" style="margin-left: 20px;">重新生成</el-button>
                <div>
                    <el-button @click="handleVisible = false">取消</el-button>
                    <el-button type="primary" @click="handleFormSubmit()">确定</el-button>
                </div>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            dataForm: {
                feedbackType: '',
                status: '',
                feedbackDesc: '',
                feedbackId: ''
            },
            dataList: [],
            pageIndex: 1,
            pageSize: 10,
            totalPage: 0,
            dataListLoading: false,
            handleVisible: false,
            handleForm: {
                feedbackId: '',
                feedbackType: '',
                feedbackTypeText: '',
                feedbackDesc: '',
                handleRemark: ''
            },
            currentRow: null, // 保存当前正在处理的行数据
            handleRule: {
                handleRemark: [
                    { required: true, message: '处理备注不能为空', trigger: 'blur' },
                    { max: 500, message: '处理备注长度不能超过500个字符', trigger: 'blur' }
                ]
            }
        }
    },
    activated() {
        // 从路由查询参数获取feedbackId
        const feedbackId = this.$route.query.feedbackId
        if (feedbackId) {
            // 如果有feedbackId，设置到查询表单中
            this.dataForm.feedbackId = feedbackId
        }
        this.getDataList()
    },
    methods: {
        // 获取数据列表
        getDataList() {
            this.dataListLoading = true
            // 获取wx_openid，从cookie或store中获取
            const wxOpenid = this.$cookie.get('wx_openid') || ''
            this.$http({
                url: this.$http.adornUrl('/manage/bizContentFeedback/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'feedbackType': this.dataForm.feedbackType,
                    'status': this.dataForm.status !== '' ? this.dataForm.status : null,
                    'feedbackDesc': this.dataForm.feedbackDesc,
                    'feedbackId': this.dataForm.feedbackId || null
                }),
                headers: {
                    'wx_openid': wxOpenid
                }
            }).then(({ data }) => {
                if (data && data.code === 200) {
                    this.dataList = data.page ? data.page.list : []
                    this.totalPage = data.page ? data.page.totalCount : 0
                } else {
                    this.dataList = []
                    this.totalPage = 0
                }
                this.dataListLoading = false
            })
        },
        // 每页数
        sizeChangeHandle(val) {
            this.pageSize = val
            this.pageIndex = 1
            this.getDataList()
        },
        // 当前页
        currentChangeHandle(val) {
            this.pageIndex = val
            this.getDataList()
        },
        // 反馈类型格式化
        feedbackTypeFormat(row, column, cellValue) {
            const typeMap = {
                'PRODUCT': '产品',
                'CHARACTER': '人物形象'
            }
            return typeMap[cellValue] || cellValue
        },
        // 状态格式化
        statusFormat(row, column, cellValue) {
            const statusMap = {
                0: '未处理',
                1: '已处理'
            }
            return statusMap[cellValue] !== undefined ? statusMap[cellValue] : '未知'
        },
        // 表格行类名
        tableRowClassName({ row, rowIndex }) {
            if (row.feedbackId) {
                return 'highlight-row'
            }
            return ''
        },
        // 处理反馈
        handleFeedback(row) {
            this.handleVisible = true
            this.currentRow = row // 保存完整的行数据
            this.$nextTick(() => {
                this.$refs['handleForm'] && this.$refs['handleForm'].resetFields()
            })
            this.handleForm = {
                feedbackId: row.feedbackId,
                feedbackType: row.feedbackType,
                feedbackTypeText: this.feedbackTypeFormat(row, null, row.feedbackType),
                feedbackDesc: row.feedbackDesc,
                handleRemark: row.handleRemark || ''
            }
        },
        // 处理反馈提交
        handleFormSubmit() {
            this.$refs['handleForm'].validate((valid) => {
                if (valid) {
                    const wxOpenid = this.$cookie.get('wx_openid') || ''
                    this.$http({
                        url: this.$http.adornUrl('/manage/bizContentFeedback/handle'),
                        method: 'post',
                        data: this.$http.adornData({
                            'feedbackId': this.handleForm.feedbackId,
                            'handleRemark': this.handleForm.handleRemark
                        }),
                        headers: {
                            'wx_openid': wxOpenid
                        }
                    }).then(({ data }) => {
                        if (data && data.code === 200) {
                            this.$message({
                                message: '处理成功',
                                type: 'success',
                                duration: 1500,
                                onClose: () => {
                                    this.handleVisible = false
                                    this.getDataList()
                                }
                            })
                        } else {
                            this.$message.error(data.msg || '处理失败')
                        }
                    })
                }
            })
        },
        // 重新生成
        regenerateHandle() {
            if (!this.currentRow) {
                this.$message.error('数据异常，请重新打开处理窗口')
                return
            }
            
            // 构建保存数据，设置 status 为 1
            const saveData = {
                feedbackId: this.currentRow.feedbackId,
                contentId: this.currentRow.contentId,
                feedbackType: this.currentRow.feedbackType,
                feedbackDesc: this.currentRow.feedbackDesc,
                userOpenid: this.currentRow.userOpenid,
                status: 1, // 设置为 1
                handleRemark: this.handleForm.handleRemark || null,
                deleted: this.currentRow.deleted !== undefined ? this.currentRow.deleted : 0,
                createTime: this.currentRow.createTime,
                updateTime: this.currentRow.updateTime
            }
            
            const wxOpenid = this.$cookie.get('wx_openid') || ''
            this.$http({
                url: this.$http.adornUrl('/manage/bizContentFeedback/save'),
                method: 'post',
                data: this.$http.adornData(saveData),
                headers: {
                    'wx_openid': wxOpenid
                }
            }).then(({ data }) => {
                if (data && data.code === 200) {
                    this.$message({
                        message: '重新生成插入成功',
                        type: 'success',
                        duration: 1500,
                        onClose: () => {
                            this.handleVisible = false
                            this.getDataList()
                        }
                    })
                } else {
                    this.$message.error(data.msg || '保存失败')
                }
            }).catch(() => {
                this.$message.error('保存失败，请稍后重试')
            })
        }
    }
}
</script>

<style scoped>
.mod-biz-feedback ::v-deep .el-table .highlight-row {
    background-color: #fffbe6;
}
.mod-biz-feedback ::v-deep .el-table .highlight-row:hover > td {
    background-color: #fffbe6 !important;
}
</style>


<template>
    <div class="mod-biz-plan-header">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.planName" placeholder="计划名称" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-select v-model="dataForm.planType" placeholder="计划类型" clearable style="width: 150px;">
                    <el-option label="私域复购" :value="1"></el-option>
                    <el-option label="公域获客" :value="2"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-select v-model="dataForm.status" placeholder="状态" clearable style="width: 120px;">
                    <el-option label="待提交" :value="0"></el-option>
                    <el-option label="已提交" :value="1"></el-option>
                    <el-option label="已执行" :value="2"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
            </el-form-item>
        </el-form>
        <el-table :data="dataList" border v-loading="dataListLoading" style="width: 100%;">
            <el-table-column prop="planName" header-align="center" align="center" label="计划名称" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column prop="planType" header-align="center" align="center" label="计划类型" :formatter="planTypeFormat" width="120">
            </el-table-column>
            <el-table-column prop="strategyType" header-align="center" align="center" label="策略类型" :formatter="strategyTypeFormat" width="120">
            </el-table-column>
            <el-table-column prop="status" header-align="center" align="center" label="状态" :formatter="statusFormat" width="100">
            </el-table-column>
            <el-table-column prop="submittedAt" header-align="center" align="center" label="提交时间" width="160">
            </el-table-column>
            <el-table-column prop="executedAt" header-align="center" align="center" label="执行时间" width="160">
            </el-table-column>
            <el-table-column prop="deliveredAt" header-align="center" align="center" label="交付时间" width="160">
            </el-table-column>
            <el-table-column prop="createTime" header-align="center" align="center" label="创建时间" width="160">
            </el-table-column>
            <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="viewPlanItems(scope.row)">查看项目</el-button>
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
    </div>
</template>

<script>
export default {
    data() {
        return {
            dataForm: {
                planName: '',
                planType: null,
                status: null
            },
            dataList: [],
            pageIndex: 1,
            pageSize: 10,
            totalPage: 0,
            dataListLoading: false
        }
    },
    activated() {
        this.getDataList()
    },
    methods: {
        // 获取数据列表
        getDataList() {
            this.dataListLoading = true
            const wxOpenid = this.$cookie.get('wx_openid') || ''
            this.$http({
                url: this.$http.adornUrl('/manage/bizPlanHeader/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'planName': this.dataForm.planName,
                    'planType': this.dataForm.planType,
                    'status': this.dataForm.status
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
        // 查看计划项目
        viewPlanItems(row) {
            this.$router.push({
                name: 'biz-planItem',
                query: {
                    planId: row.planId,
                    planName: row.planName
                }
            })
        },
        // 计划类型格式化
        planTypeFormat(row, column, cellValue) {
            const typeMap = {
                1: '私域复购',
                2: '公域获客'
            }
            return typeMap[cellValue] || '未知'
        },
        // 策略类型格式化
        strategyTypeFormat(row, column, cellValue) {
            const typeMap = {
                1: '私域复购',
                2: '公域获客'
            }
            return typeMap[cellValue] || '未知'
        },
        // 状态格式化
        statusFormat(row, column, cellValue) {
            const statusMap = {
                0: '待提交',
                1: '已提交',
                2: '已执行'
            }
            return statusMap[cellValue] !== undefined ? statusMap[cellValue] : '未知'
        }
    }
}
</script>


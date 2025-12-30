<template>
    <div class="mod-biz-store">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.storeName" placeholder="店铺名称" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
                <el-button v-if="isAuth('wx:manage:bizStore:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
            </el-form-item>
        </el-form>
        <el-table :data="dataList" border v-loading="dataListLoading" style="width: 100%;">
            <el-table-column prop="storeName" header-align="center" align="center" label="店铺名称">
            </el-table-column>
            <el-table-column prop="ownerName" header-align="center" align="center" label="店主姓名">
            </el-table-column>
            <el-table-column prop="ownerPhone" header-align="center" align="center" label="联系电话">
            </el-table-column>
            <el-table-column header-align="center" align="center" label="销售" width="120">
                <template slot-scope="scope">
                    <span>{{ scope.row.salesId || scope.row.sales_id || '-' }}</span>
                </template>
            </el-table-column>
            <el-table-column prop="cateringType" header-align="center" align="center" label="餐饮类型" :formatter="cateringTypeFormat">
            </el-table-column>
            <el-table-column prop="storeAddress" header-align="center" align="center" label="店铺地址" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column prop="customerGroup" header-align="center" align="center" label="客户群体" :formatter="customerGroupFormat">
            </el-table-column>
            <el-table-column prop="auditStatus" header-align="center" align="center" label="审核状态" :formatter="auditStatusFormat">
            </el-table-column>
            <el-table-column header-align="center" align="center" label="周计划" width="100">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="goToPlanItem(scope.row)">查看</el-button>
                </template>
            </el-table-column>
            <el-table-column prop="createTime" header-align="center" align="center" label="创建时间">
            </el-table-column>
            <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row)">修改</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 弹窗, 新增 / 修改 -->
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    </div>
</template>

<script>
import AddOrUpdate from './add-or-update'
export default {
    data() {
        return {
            dataForm: {
                storeName: '',
                salesId: ''
            },
            dataList: [],
            dataListLoading: false,
            addOrUpdateVisible: false
        }
    },
    components: {
        AddOrUpdate
    },
    activated() {
        // 从路由参数中获取salesId
        if (this.$route.query.salesId) {
            this.dataForm.salesId = this.$route.query.salesId
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
                url: this.$http.adornUrl('/manage/bizStore/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'storeName': this.dataForm.storeName,
                    'salesId': this.dataForm.salesId
                }),
                headers: {
                    'wx_openid': wxOpenid
                }
            }).then(({ data }) => {
                if (data && data.code === 200) {
                    this.dataList = data.page ? data.page.list : []
                } else {
                    this.dataList = []
                }
                this.dataListLoading = false
            })
        },
        // 新增 / 修改
        addOrUpdateHandle(item) {
            this.addOrUpdateVisible = true
            this.$nextTick(() => {
                this.$refs.addOrUpdate.init(item)
            })
        },
        // 餐饮类型格式化
        cateringTypeFormat(row, column, cellValue) {
            const typeMap = {
                'CHINESE': '中餐',
                'WESTERN': '西餐',
                'JAPANESE': '日料',
                'KOREAN': '韩餐',
                'SNACK': '小吃',
                'DESSERT': '甜品',
                'BEVERAGE': '饮品',
                'OTHER': '其他'
            }
            return typeMap[cellValue] || cellValue
        },
        // 客户群体格式化
        customerGroupFormat(row, column, cellValue) {
            const groupMap = {
                'OFFICE_WORKER': '上班族',
                'STUDENT': '学生',
                'FAMILY': '家庭',
                'TOURIST': '游客',
                'OTHER': '其他'
            }
            return groupMap[cellValue] || cellValue
        },
        // 审核状态格式化
        auditStatusFormat(row, column, cellValue) {
            const statusMap = {
                0: '待审核',
                1: '已通过',
                2: '已拒绝'
            }
            return statusMap[cellValue] !== undefined ? statusMap[cellValue] : '未知'
        },
        // 跳转到周计划表页面
        goToPlanItem(row) {
            const storeId = row.storeId || row.store_id
            if (!storeId) {
                this.$message.warning('店铺ID不存在')
                return
            }
            this.$router.push({
                name: 'manage-bizPlanHeader',
                query: {
                    storeId: storeId
                }
            })
        }
    }
}
</script>


<template>
    <div class="mod-biz-plan-item">
        <!-- 周计划信息卡片 -->
        <el-card class="plan-header-card" shadow="never" style="margin-bottom: 20px;">
            <div slot="header" class="clearfix">
                <span style="font-weight: bold; font-size: 16px;">周计划信息</span>
            </div>
            <el-row :gutter="20" v-loading="planHeaderLoading">
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">计划名称：</span>
                        <span class="info-value">{{ planHeaderInfo.planName || '-' }}</span>
                    </div>
                </el-col>
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">计划类型：</span>
                        <span class="info-value">{{ planTypeFormat(null, null, planHeaderInfo.planType) }}</span>
                    </div>
                </el-col>
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">策略类型：</span>
                        <span class="info-value">{{ strategyTypeFormat(null, null, planHeaderInfo.strategyType) }}</span>
                    </div>
                </el-col>
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">状态：</span>
                        <el-tag :type="getStatusTagType(planHeaderInfo.status)" size="small">
                            {{ statusFormat(null, null, planHeaderInfo.status) }}
                        </el-tag>
                    </div>
                </el-col>
            </el-row>
            <el-row :gutter="20" style="margin-top: 15px;">
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">提交时间：</span>
                        <span class="info-value">{{ planHeaderInfo.submittedAt || '-' }}</span>
                    </div>
                </el-col>
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">执行时间：</span>
                        <span class="info-value">{{ planHeaderInfo.executedAt || '-' }}</span>
                    </div>
                </el-col>
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">交付时间：</span>
                        <span class="info-value">{{ planHeaderInfo.deliveredAt || '-' }}</span>
                    </div>
                </el-col>
                <el-col :span="6">
                    <div class="info-item">
                        <span class="info-label">创建时间：</span>
                        <span class="info-value">{{ planHeaderInfo.createTime || '-' }}</span>
                    </div>
                </el-col>
            </el-row>
        </el-card>
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.productName" placeholder="产品名称" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-select v-model="dataForm.platform" placeholder="平台" clearable style="width: 150px;">
                    <el-option label="抖音" value="DOUYIN"></el-option>
                    <el-option label="小红书" value="XIAOHONGSHU"></el-option>
                    <el-option label="美团" value="MEITUAN"></el-option>
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
            <el-table-column prop="dateKey" header-align="center" align="center" label="周期" width="100">
            </el-table-column>
            <el-table-column prop="timeSlot" header-align="center" align="center" label="时段" width="100" :formatter="timeSlotFormat">
            </el-table-column>
            <el-table-column prop="platformDesc" header-align="center" align="center" label="平台" width="120">
            </el-table-column>
            <el-table-column prop="contentTagDesc" header-align="center" align="center" label="内容标签" width="120">
            </el-table-column>
            <el-table-column prop="marketingTheme" header-align="center" align="center" label="营销主题" width="120" :formatter="marketingThemeFormat">
            </el-table-column>
            <el-table-column prop="productName" header-align="center" align="center" label="产品名称" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column prop="originalPrice" header-align="center" align="center" label="原价" width="100">
                <template slot-scope="scope">
                    <span v-if="scope.row.originalPrice">{{ scope.row.originalPrice.toFixed(2) }}</span>
                    <span v-else>-</span>
                </template>
            </el-table-column>
            <el-table-column prop="discountPrice" header-align="center" align="center" label="折扣价" width="100">
                <template slot-scope="scope">
                    <span v-if="scope.row.discountPrice">{{ scope.row.discountPrice.toFixed(2) }}</span>
                    <span v-else>-</span>
                </template>
            </el-table-column>
            <el-table-column prop="activityDetails" header-align="center" align="center" label="活动详情" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column prop="status" header-align="center" align="center" label="状态" :formatter="statusFormat" width="100">
            </el-table-column>
            <el-table-column prop="submittedAt" header-align="center" align="center" label="提交时间" width="160">
            </el-table-column>
            <el-table-column prop="executedAt" header-align="center" align="center" label="执行时间" width="160">
            </el-table-column>
            <el-table-column prop="deliveredAt" header-align="center" align="center" label="交付时间" width="160">
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
                productName: '',
                platform: '',
                status: null
            },
            dataList: [],
            pageIndex: 1,
            pageSize: 10,
            totalPage: 0,
            dataListLoading: false,
            planId: '',
            planHeaderInfo: {},
            planHeaderLoading: false
        }
    },
    activated() {
        // 从路由参数获取planId
        this.planId = this.$route.query.planId || ''
        if (!this.planId) {
            this.$message.error('缺少计划ID参数')
            this.$router.go(-1)
            return
        }
        this.getPlanHeaderInfo()
        this.getDataList()
    },
    methods: {
        // 获取周计划信息
        getPlanHeaderInfo() {
            if (!this.planId) {
                return
            }
            this.planHeaderLoading = true
            const wxOpenid = this.$cookie.get('wx_openid') || ''
            // 获取列表，然后在前端筛选匹配的planId
            this.$http({
                url: this.$http.adornUrl('/manage/bizPlanHeader/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': 1,
                    'limit': 100
                }),
                headers: {
                    'wx_openid': wxOpenid
                }
            }).then(({ data }) => {
                if (data && data.code === 200 && data.page && data.page.list && data.page.list.length > 0) {
                    // 从列表中查找匹配的planId
                    const plan = data.page.list.find(item => item.planId === this.planId)
                    if (plan) {
                        this.planHeaderInfo = plan
                    } else {
                        // 如果找不到，使用路由参数中的planName
                        this.planHeaderInfo = {
                            planId: this.planId,
                            planName: this.$route.query.planName || ''
                        }
                    }
                } else {
                    // 如果API返回空，使用路由参数
                    this.planHeaderInfo = {
                        planId: this.planId,
                        planName: this.$route.query.planName || ''
                    }
                }
                this.planHeaderLoading = false
            }).catch(() => {
                // 如果API调用失败，使用路由参数
                this.planHeaderInfo = {
                    planId: this.planId,
                    planName: this.$route.query.planName || ''
                }
                this.planHeaderLoading = false
            })
        },
        // 获取数据列表
        getDataList() {
            if (!this.planId) {
                return
            }
            this.dataListLoading = true
            const wxOpenid = this.$cookie.get('wx_openid') || ''
            this.$http({
                url: this.$http.adornUrl('/manage/bizPlanItem/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'planId': this.planId,
                    'productName': this.dataForm.productName,
                    'platform': this.dataForm.platform,
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
        // 时段格式化
        timeSlotFormat(row, column, cellValue) {
            if (!cellValue) return '-'
            const timeSlotMap = {
                'ALL': '全天',
                'MORNING': '上午',
                'AFTERNOON': '下午',
                'EVENING': '晚上'
            }
            return timeSlotMap[cellValue] || cellValue
        },
        // 营销主题格式化
        marketingThemeFormat(row, column, cellValue) {
            if (!cellValue) return '-'
            const themeMap = {
                'MEMBER': '会员',
                'PROMOTION': '促销',
                'NEW_PRODUCT': '新品',
                'SEASONAL': '季节'
            }
            return themeMap[cellValue] || cellValue
        },
        // 状态格式化
        statusFormat(row, column, cellValue) {
            const statusMap = {
                0: '待提交',
                1: '已提交',
                2: '已执行'
            }
            return statusMap[cellValue] !== undefined ? statusMap[cellValue] : '未知'
        },
        // 计划类型格式化
        planTypeFormat(row, column, cellValue) {
            if (cellValue === null || cellValue === undefined) return '-'
            const typeMap = {
                1: '私域复购',
                2: '公域获客'
            }
            return typeMap[cellValue] || '未知'
        },
        // 策略类型格式化
        strategyTypeFormat(row, column, cellValue) {
            if (cellValue === null || cellValue === undefined) return '-'
            const typeMap = {
                1: '私域复购',
                2: '公域获客'
            }
            return typeMap[cellValue] || '未知'
        },
        // 获取状态标签类型
        getStatusTagType(status) {
            const typeMap = {
                0: 'info',
                1: 'warning',
                2: 'success'
            }
            return typeMap[status] || ''
        }
    }
}
</script>

<style lang="scss" scoped>
.plan-header-card {
    .info-item {
        margin-bottom: 10px;
        .info-label {
            color: #606266;
            font-size: 14px;
        }
        .info-value {
            color: #303133;
            font-size: 14px;
            font-weight: 500;
        }
    }
}
</style>


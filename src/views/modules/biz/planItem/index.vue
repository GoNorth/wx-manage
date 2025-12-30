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
            <el-table-column prop="rsThumbnailUrl" header-align="center" align="center" label="图片" width="120">
                <template slot-scope="scope">
                    <div v-if="scope.row.rsThumbnailUrl" class="image-cell">
                        <!-- 视频类型 (content_type === 2) -->
                        <div
                            v-if="scope.row.rsFileUrl && (scope.row.contentType === 2 || scope.row.content_type === 2)"
                            class="video-thumbnail-wrapper"
                            @click.stop="openVideoDialog(scope.row.rsFileUrl)"
                        >
                            <el-popover
                                placement="right"
                                trigger="hover"
                                :open-delay="300"
                                width="500"
                            >
                                <video
                                    :src="scope.row.rsFileUrl"
                                    controls
                                    style="max-width: 500px; max-height: 500px;"
                                    @click.stop
                                ></video>
                                <div slot="reference" class="video-thumbnail-container">
                                    <el-image
                                        :src="scope.row.rsThumbnailUrl"
                                        fit="cover"
                                        style="width: 80px; height: 80px; cursor: pointer;"
                                        class="thumbnail-image"
                                    >
                                    </el-image>
                                </div>
                            </el-popover>
                        </div>
                        <!-- 图片类型 (content_type === 1) 或者没有contentType但有rsFileUrl时默认当作图片 -->
                        <div
                            v-else-if="scope.row.rsFileUrl"
                            class="image-click-wrapper"
                            @click.stop="openImageDialog(scope.row.rsFileUrl)"
                        >
                            <el-image
                                :src="scope.row.rsThumbnailUrl"
                                fit="cover"
                                style="width: 80px; height: 80px; cursor: pointer;"
                                class="thumbnail-image"
                            >
                            </el-image>
                        </div>
                        <!-- 只有缩略图，没有文件URL -->
                        <el-image
                            v-else
                            :src="scope.row.rsThumbnailUrl"
                            fit="cover"
                            style="width: 80px; height: 80px;"
                            class="thumbnail-image"
                        >
                        </el-image>
                    </div>
                    <span v-else>-</span>
                </template>
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
        <!-- 图片预览对话框 -->
        <el-dialog
            title="图片预览"
            :visible.sync="imageDialogVisible"
            :width="imageDialogWidth"
            :close-on-click-modal="true"
            custom-class="image-preview-dialog"
        >
            <div class="image-preview-container">
                <img
                    v-if="currentImageUrl"
                    :src="currentImageUrl"
                    class="preview-image"
                    alt="预览图片"
                />
            </div>
        </el-dialog>
        <!-- 视频预览对话框 -->
        <el-dialog
            title="视频预览"
            :visible.sync="videoDialogVisible"
            :width="videoDialogWidth"
            :close-on-click-modal="true"
            custom-class="video-preview-dialog"
        >
            <div class="video-preview-container">
                <video
                    v-if="currentVideoUrl"
                    :key="currentVideoUrl"
                    :src="currentVideoUrl"
                    controls
                    preload="auto"
                    playsinline
                    webkit-playsinline
                    x5-playsinline
                    class="preview-video"
                    @error="handleVideoError"
                    @loadedmetadata="handleVideoLoaded"
                >
                    您的浏览器不支持视频播放。
                </video>
            </div>
        </el-dialog>
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
            storeId: '',
            planHeaderInfo: {},
            planHeaderLoading: false,
            imageDialogVisible: false,
            currentImageUrl: '',
            videoDialogVisible: false,
            currentVideoUrl: ''
        }
    },
    computed: {
        // 图片对话框宽度：根据9:16比例计算
        imageDialogWidth() {
            // 9:16比例，如果最大高度是90vh，宽度应该是 90vh * 9/16 = 50.625vh
            return '50.625vh'
        },
        // 视频对话框宽度：根据9:16比例计算
        videoDialogWidth() {
            // 视频也使用9:16比例
            return '50.625vh'
        }
    },
    activated() {
        // 从路由参数获取planId或storeId
        this.planId = this.$route.query.planId || ''
        this.storeId = this.$route.query.storeId || ''
        if (!this.planId && !this.storeId) {
            this.$message.error('缺少计划ID或店铺ID参数')
            this.$router.go(-1)
            return
        }
        if (this.planId) {
            this.getPlanHeaderInfo()
        }
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
            if (!this.planId && !this.storeId) {
                return
            }
            this.dataListLoading = true
            const wxOpenid = this.$cookie.get('wx_openid') || ''
            const params = {
                'page': this.pageIndex,
                'limit': this.pageSize,
                'productName': this.dataForm.productName,
                'platform': this.dataForm.platform,
                'status': this.dataForm.status
            }
            // 如果提供了planId，使用planId；如果提供了storeId，使用storeId
            if (this.planId) {
                params.planId = this.planId
            }
            if (this.storeId) {
                params.storeId = this.storeId
            }
            this.$http({
                url: this.$http.adornUrl('/manage/bizPlanItem/list'),
                method: 'get',
                params: this.$http.adornParams(params),
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
        },
        // 打开图片预览对话框
        openImageDialog(imageUrl) {
            console.log('openImageDialog called with:', imageUrl)
            if (!imageUrl) {
                this.$message.warning('图片地址不存在')
                return
            }
            this.currentImageUrl = imageUrl
            this.imageDialogVisible = true
        },
        // 打开视频预览对话框
        openVideoDialog(videoUrl) {
            console.log('openVideoDialog called with:', videoUrl)
            if (!videoUrl) {
                this.$message.warning('视频地址不存在')
                return
            }
            // 先关闭对话框（如果已打开），然后重新打开，确保视频重新加载
            if (this.videoDialogVisible) {
                this.videoDialogVisible = false
                this.$nextTick(() => {
                    this.currentVideoUrl = videoUrl
                    this.videoDialogVisible = true
                })
            } else {
                this.currentVideoUrl = videoUrl
                this.videoDialogVisible = true
            }
        },
        // 处理视频加载错误
        handleVideoError(e) {
            console.error('视频加载失败:', e)
            const video = e.target
            console.error('视频错误详情:', {
                error: video.error,
                networkState: video.networkState,
                readyState: video.readyState,
                src: video.src
            })
            this.$message.error('视频加载失败，请检查视频地址是否正确或视频格式是否支持')
        },
        // 处理视频加载成功
        handleVideoLoaded(e) {
            console.log('视频加载成功:', e.target.duration, '秒')
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
.image-cell {
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    .thumbnail-image {
        border-radius: 4px;
        transition: transform 0.3s;
        &:hover {
            transform: scale(1.1);
            box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
        }
    }
    .image-click-wrapper {
        cursor: pointer;
        display: inline-block;
    }
    .video-thumbnail-wrapper {
        position: relative;
        display: inline-block;
        .video-thumbnail-container {
            position: relative;
            display: inline-block;
            cursor: pointer;
        }
    }
}
.image-preview-container {
    display: block;
    margin: 0;
    padding: 0;
    width: 100%;
    aspect-ratio: 9 / 16;
    position: relative;
    .preview-image {
        width: 100%;
        height: 100%;
        object-fit: contain;
        display: block;
        margin: 0;
    }
}
::v-deep .image-preview-dialog {
    .el-dialog {
        margin: 0 auto !important;
        margin-top: 10px !important;
        display: flex;
        flex-direction: column;
        aspect-ratio: 9 / 16;
        max-height: calc(100vh - 20px);
    }
    .el-dialog__body {
        padding: 0 !important;
        margin: 0;
        width: 100%;
        flex: 1;
        overflow: hidden;
        aspect-ratio: 9 / 16;
    }
    .el-dialog__header {
        padding: 10px 20px;
        margin: 0;
        flex-shrink: 0;
    }
    .el-dialog__headerbtn {
        top: 10px;
        right: 20px;
    }
}
.video-preview-container {
    display: block;
    margin: 0;
    padding: 0;
    width: 100%;
    aspect-ratio: 9 / 16;
    position: relative;
    .preview-video {
        width: 100%;
        height: 100%;
        object-fit: contain;
        display: block;
        margin: 0;
    }
}
::v-deep .video-preview-dialog {
    .el-dialog {
        margin: 0 auto !important;
        margin-top: 10px !important;
        display: flex;
        flex-direction: column;
        aspect-ratio: 9 / 16;
        max-height: calc(100vh - 20px);
    }
    .el-dialog__body {
        padding: 0 !important;
        margin: 0;
        width: 100%;
        flex: 1;
        overflow: hidden;
        aspect-ratio: 9 / 16;
    }
    .el-dialog__header {
        padding: 10px 20px;
        margin: 0;
        flex-shrink: 0;
    }
    .el-dialog__headerbtn {
        top: 10px;
        right: 20px;
    }
}
</style>


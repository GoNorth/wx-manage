<template>
    <div class="mod-biz-sales">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.salesAccount" placeholder="销售账号" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-input v-model="dataForm.salesName" placeholder="销售姓名" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-select v-model="dataForm.gender" placeholder="性别" clearable style="width: 120px;">
                    <el-option label="男" value="MALE"></el-option>
                    <el-option label="女" value="FEMALE"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-input v-model="dataForm.phone" placeholder="手机号" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
                <el-button v-if="isAuth('wx:manage:bizSales:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
            </el-form-item>
        </el-form>
        <el-table :data="dataList" border v-loading="dataListLoading" style="width: 100%;">
            <el-table-column prop="salesAccount" header-align="center" align="center" label="销售账号" width="120">
            </el-table-column>
            <el-table-column prop="salesName" header-align="center" align="center" label="销售姓名" width="120">
            </el-table-column>
            <el-table-column prop="gender" header-align="center" align="center" label="性别" width="80" :formatter="genderFormat">
            </el-table-column>
            <el-table-column prop="age" header-align="center" align="center" label="年龄" width="80">
            </el-table-column>
            <el-table-column prop="phone" header-align="center" align="center" label="手机号" width="130">
            </el-table-column>
            <el-table-column prop="address" header-align="center" align="center" label="住址" :show-overflow-tooltip="true">
            </el-table-column>
            <el-table-column header-align="center" align="center" label="二维码" width="100">
                <template slot-scope="scope">
                    <el-popover
                        v-if="scope.row.qrcodeUrl"
                        placement="right"
                        trigger="hover"
                        :open-delay="300"
                        width="300"
                    >
                        <el-image
                            :src="scope.row.qrcodeUrl"
                            fit="contain"
                            style="max-width: 300px; max-height: 300px;"
                            :preview-src-list="[scope.row.qrcodeUrl]"
                        >
                            <div slot="error" class="image-slot">
                                <i class="el-icon-picture-outline"></i>
                                <p>图片加载失败</p>
                            </div>
                        </el-image>
                        <span slot="reference" style="color: #409EFF; cursor: pointer;">二维码</span>
                    </el-popover>
                    <span v-else>-</span>
                </template>
            </el-table-column>
            <el-table-column header-align="center" align="center" label="门店" width="100">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="viewStores(scope.row.salesId)">查看门店</el-button>
                </template>
            </el-table-column>
            <el-table-column prop="createTime" header-align="center" align="center" label="创建时间" width="160">
            </el-table-column>
            <el-table-column prop="updateTime" header-align="center" align="center" label="更新时间" width="160">
            </el-table-column>
            <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row)">修改</el-button>
                    <el-button type="text" size="small" style="color: #f56c6c;" @click="deleteHandle(scope.row.salesId)">删除</el-button>
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
                salesAccount: '',
                salesName: '',
                gender: '',
                phone: ''
            },
            dataList: [],
            pageIndex: 1,
            pageSize: 10,
            totalPage: 0,
            dataListLoading: false,
            addOrUpdateVisible: false
        }
    },
    components: {
        AddOrUpdate
    },
    activated() {
        this.getDataList()
    },
    methods: {
        // 获取数据列表
        getDataList() {
            this.dataListLoading = true
            // 获取wx_openid，从cookie或store中获取
            const wxOpenid = this.$cookie.get('wx_openid') || ''
            this.$http({
                url: this.$http.adornUrl('/manage/bizSales/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'salesAccount': this.dataForm.salesAccount,
                    'salesName': this.dataForm.salesName,
                    'gender': this.dataForm.gender,
                    'phone': this.dataForm.phone
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
        // 新增 / 修改
        addOrUpdateHandle(item) {
            this.addOrUpdateVisible = true
            this.$nextTick(() => {
                this.$refs.addOrUpdate.init(item)
            })
        },
        // 删除
        deleteHandle(salesId) {
            this.$confirm(`确定对[id=${salesId}]进行[删除]操作?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                const wxOpenid = this.$cookie.get('wx_openid') || ''
                this.$http({
                    url: this.$http.adornUrl(`/manage/bizSales/delete`),
                    method: 'post',
                    data: this.$http.adornData({
                        'salesId': salesId
                    }),
                    headers: {
                        'wx_openid': wxOpenid
                    }
                }).then(({ data }) => {
                    if (data && data.code === 200) {
                        this.$message({
                            message: '操作成功',
                            type: 'success',
                            duration: 1500,
                            onClose: () => {
                                this.getDataList()
                            }
                        })
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            }).catch(() => {})
        },
        // 性别格式化
        genderFormat(row, column, cellValue) {
            const genderMap = {
                'MALE': '男',
                'FEMALE': '女'
            }
            return genderMap[cellValue] || cellValue
        },
        // 查看门店
        viewStores(salesId) {
            this.$router.push({
                name: 'manage-bizStore',
                query: {
                    salesId: salesId
                }
            })
        }
    }
}
</script>


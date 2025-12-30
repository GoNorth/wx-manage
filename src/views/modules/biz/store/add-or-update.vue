<template>
    <el-dialog :title="!dataForm.storeId ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible" width="50%">
        <el-tabs v-model="activeTab" type="card">
            <!-- 基础信息 -->
            <el-tab-pane label="基础信息" name="basic">
                <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="120px" style="margin-top: 20px;">
                    <el-row :gutter="20">
                        <el-col :span="12">
                            <el-form-item label="店铺名称" prop="storeName">
                                <el-input v-model="dataForm.storeName" placeholder="店铺名称"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="餐饮类型" prop="cateringType">
                                <el-select v-model="dataForm.cateringType" placeholder="请选择餐饮类型" style="width: 100%;">
                                    <el-option label="中餐" value="CHINESE"></el-option>
                                    <el-option label="西餐" value="WESTERN"></el-option>
                                    <el-option label="日料" value="JAPANESE"></el-option>
                                    <el-option label="韩餐" value="KOREAN"></el-option>
                                    <el-option label="小吃" value="SNACK"></el-option>
                                    <el-option label="甜品" value="DESSERT"></el-option>
                                    <el-option label="饮品" value="BEVERAGE"></el-option>
                                    <el-option label="其他" value="OTHER"></el-option>
                                </el-select>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-row :gutter="20">
                        <el-col :span="12">
                            <el-form-item label="店主姓名" prop="ownerName">
                                <el-input v-model="dataForm.ownerName" placeholder="店主姓名"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="联系电话" prop="ownerPhone">
                                <el-input v-model="dataForm.ownerPhone" placeholder="联系电话"></el-input>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-form-item label="店铺地址" prop="storeAddress">
                        <el-input v-model="dataForm.storeAddress" placeholder="店铺地址" type="textarea" :rows="2"></el-input>
                    </el-form-item>
                    <el-row :gutter="20">
                        <el-col :span="12">
                            <el-form-item label="经度" prop="longitude">
                                <el-input-number v-model="dataForm.longitude" :precision="7" :step="0.0000001" style="width: 100%;"></el-input-number>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="纬度" prop="latitude">
                                <el-input-number v-model="dataForm.latitude" :precision="7" :step="0.0000001" style="width: 100%;"></el-input-number>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-form-item label="客户群体" prop="customerGroup">
                        <el-select v-model="dataForm.customerGroup" placeholder="请选择客户群体" style="width: 100%;">
                            <el-option label="上班族" value="OFFICE_WORKER"></el-option>
                            <el-option label="学生" value="STUDENT"></el-option>
                            <el-option label="家庭" value="FAMILY"></el-option>
                            <el-option label="游客" value="TOURIST"></el-option>
                            <el-option label="其他" value="OTHER"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="审核状态" prop="auditStatus" v-if="dataForm.storeId">
                        <el-radio-group v-model="dataForm.auditStatus">
                            <el-radio :label="0">待审核</el-radio>
                            <el-radio :label="1">已通过</el-radio>
                            <el-radio :label="2">已拒绝</el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="审核备注" prop="auditRemark" v-if="dataForm.storeId">
                        <el-input v-model="dataForm.auditRemark" placeholder="审核备注" type="textarea" :rows="3"></el-input>
                    </el-form-item>
                </el-form>
            </el-tab-pane>
            
            <!-- 人物形象 -->
            <el-tab-pane label="人物形象" name="character">
                <el-form :model="dataForm" ref="characterForm" label-width="120px" style="margin-top: 20px;">
                    <el-form-item label="人物头像">
                        <div style="display: flex; align-items: center;">
                            <img v-if="dataForm.characterAvatar" :src="dataForm.characterAvatar" class="avatar-preview" style="width: 100px; height: 100px; margin-right: 10px; border: 1px solid #dcdfe6; border-radius: 4px;">
                            <el-input v-model="dataForm.characterAvatar" placeholder="人物头像URL" style="flex: 1;">
                                <OssUploader slot="append" @uploaded="dataForm.characterAvatar=$event">上传</OssUploader>
                            </el-input>
                        </div>
                        <div class="el-upload__tip">支持jpg/png格式，建议尺寸200x200px</div>
                    </el-form-item>
                    <el-form-item label="人物名称">
                        <el-input v-model="dataForm.characterName" placeholder="人物名称"></el-input>
                    </el-form-item>
                    <el-form-item label="人物介绍">
                        <el-input v-model="dataForm.characterDesc" placeholder="人物介绍" type="textarea" :rows="5"></el-input>
                    </el-form-item>
                    <el-form-item label="人物形象图片">
                        <div v-for="(img, index) in characterImageList" :key="index" style="display: inline-block; margin-right: 10px; margin-bottom: 10px; position: relative;">
                            <img :src="img.url || img" @click="handlePictureCardPreview({url: img.url || img})" style="width: 100px; height: 100px; border: 1px solid #dcdfe6; border-radius: 4px; cursor: pointer;">
                            <i class="el-icon-close" @click="removeCharacterImage(index)" style="position: absolute; top: -8px; right: -8px; cursor: pointer; background: #fff; border-radius: 50%; padding: 2px;"></i>
                        </div>
                        <div style="display: inline-block; vertical-align: top;">
                            <OssUploader @uploaded="onCharacterImageUploaded"></OssUploader>
                        </div>
                        <el-dialog :visible.sync="dialogVisible" :append-to-body="true">
                            <img width="100%" :src="dialogImageUrl" alt="">
                        </el-dialog>
                    </el-form-item>
                </el-form>
            </el-tab-pane>
            
            <!-- 店铺VI -->
            <el-tab-pane label="店铺VI" name="vi">
                <el-form :model="dataForm" ref="viForm" label-width="120px" style="margin-top: 20px;">
                    <el-form-item label="店铺Logo">
                        <div style="display: flex; align-items: center;">
                            <img v-if="dataForm.storeLogo" :src="dataForm.storeLogo" class="avatar-preview" style="width: 100px; height: 100px; margin-right: 10px; border: 1px solid #dcdfe6; border-radius: 4px;">
                            <el-input v-model="dataForm.storeLogo" placeholder="店铺Logo URL" style="flex: 1;">
                                <OssUploader slot="append" @uploaded="dataForm.storeLogo=$event">上传</OssUploader>
                            </el-input>
                        </div>
                        <div class="el-upload__tip">支持jpg/png格式，建议尺寸200x200px</div>
                    </el-form-item>
                    <el-form-item label="店铺主色调">
                        <el-color-picker v-model="dataForm.mainColor"></el-color-picker>
                        <span style="margin-left: 10px;">{{ dataForm.mainColor || '未设置' }}</span>
                    </el-form-item>
                    <el-form-item label="店铺VI图片">
                        <div v-for="(img, index) in viImageList" :key="index" style="display: inline-block; margin-right: 10px; margin-bottom: 10px; position: relative;">
                            <img :src="img.url || img" @click="handlePictureCardPreview({url: img.url || img})" style="width: 100px; height: 100px; border: 1px solid #dcdfe6; border-radius: 4px; cursor: pointer;">
                            <i class="el-icon-close" @click="removeViImage(index)" style="position: absolute; top: -8px; right: -8px; cursor: pointer; background: #fff; border-radius: 50%; padding: 2px;"></i>
                        </div>
                        <div style="display: inline-block; vertical-align: top;">
                            <OssUploader @uploaded="onViImageUploaded"></OssUploader>
                        </div>
                    </el-form-item>
                    <el-form-item label="店铺介绍">
                        <el-input v-model="dataForm.storeDesc" placeholder="店铺介绍" type="textarea" :rows="5"></el-input>
                    </el-form-item>
                </el-form>
            </el-tab-pane>
        </el-tabs>
        <span slot="footer" class="dialog-footer">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
        </span>
    </el-dialog>
</template>

<script>
import OssUploader from '../../oss/oss-uploader'
export default {
    components: {
        OssUploader
    },
    data() {
        return {
            visible: false,
            activeTab: 'basic',
            dialogVisible: false,
            dialogImageUrl: '',
            characterImageList: [],
            viImageList: [],
            dataForm: {
                storeId: '',
                ownerOpenid: '',
                ownerName: '',
                ownerPhone: '',
                storeName: '',
                cateringType: '',
                storeAddress: '',
                longitude: null,
                latitude: null,
                customerGroup: '',
                auditStatus: 0,
                auditRemark: '',
                // 人物形象相关
                characterAvatar: '',
                characterName: '',
                characterDesc: '',
                characterImages: [],
                // 店铺VI相关
                storeLogo: '',
                mainColor: '',
                viImages: [],
                storeDesc: ''
            },
            dataRule: {
                storeName: [
                    { required: true, message: '店铺名称不能为空', trigger: 'blur' }
                ],
                ownerName: [
                    { required: true, message: '店主姓名不能为空', trigger: 'blur' }
                ],
                ownerPhone: [
                    { required: true, message: '联系电话不能为空', trigger: 'blur' },
                    { pattern: /^1[3-9]\d{9}$/, message: '请输入正确的手机号码', trigger: 'blur' }
                ],
                storeAddress: [
                    { required: true, message: '店铺地址不能为空', trigger: 'blur' }
                ],
                cateringType: [
                    { required: true, message: '请选择餐饮类型', trigger: 'change' }
                ],
                customerGroup: [
                    { required: true, message: '请选择客户群体', trigger: 'change' }
                ]
            }
        }
    },
    methods: {
        init(item) {
            this.visible = true
            this.activeTab = 'basic'
            this.$nextTick(() => {
                this.$refs['dataForm'] && this.$refs['dataForm'].resetFields()
            })
            
            if (item && item.storeId) {
                // 修改模式，加载数据
                this.$http({
                    url: this.$http.adornUrl(`/manage/bizStore/info/${item.storeId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({ data }) => {
                    if (data && data.code === 200) {
                        Object.assign(this.dataForm, data.bizStore || {})
                        // 处理图片列表
                        if (data.bizStore.characterImages) {
                            this.characterImageList = Array.isArray(data.bizStore.characterImages) 
                                ? data.bizStore.characterImages.map((url, index) => ({ url, uid: index, name: `character_${index}.jpg` }))
                                : []
                        }
                        if (data.bizStore.viImages) {
                            this.viImageList = Array.isArray(data.bizStore.viImages)
                                ? data.bizStore.viImages.map((url, index) => ({ url, uid: index, name: `vi_${index}.jpg` }))
                                : []
                        }
                    }
                })
            } else {
                // 新增模式
                this.dataForm = {
                    storeId: '',
                    ownerOpenid: this.$cookie.get('wx_openid') || '',
                    ownerName: '',
                    ownerPhone: '',
                    storeName: '',
                    cateringType: '',
                    storeAddress: '',
                    longitude: null,
                    latitude: null,
                    customerGroup: '',
                    auditStatus: 0,
                    auditRemark: '',
                    characterAvatar: '',
                    characterName: '',
                    characterDesc: '',
                    characterImages: [],
                    storeLogo: '',
                    mainColor: '',
                    viImages: [],
                    storeDesc: ''
                }
                this.characterImageList = []
                this.viImageList = []
            }
        },
        // 表单提交
        dataFormSubmit() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    // 收集图片URL
                    this.dataForm.characterImages = this.characterImageList.map(item => item.url || item.response?.url).filter(Boolean)
                    this.dataForm.viImages = this.viImageList.map(item => item.url || item.response?.url).filter(Boolean)
                    
                    const wxOpenid = this.$cookie.get('wx_openid') || ''
                    this.$http({
                        url: this.$http.adornUrl(`/manage/bizStore/${!this.dataForm.storeId ? 'save' : 'update'}`),
                        method: 'post',
                        data: this.$http.adornData(this.dataForm),
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
                                    this.visible = false
                                    this.$emit('refreshDataList')
                                }
                            })
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                } else {
                    // 切换到基础信息标签页
                    this.activeTab = 'basic'
                }
            })
        },
        // 人物形象图片上传完成
        onCharacterImageUploaded(url) {
            if (url) {
                this.characterImageList.push({ url: url })
            }
        },
        // 删除人物形象图片
        removeCharacterImage(index) {
            this.characterImageList.splice(index, 1)
        },
        // 店铺VI图片上传完成
        onViImageUploaded(url) {
            if (url) {
                this.viImageList.push({ url: url })
            }
        },
        // 删除店铺VI图片
        removeViImage(index) {
            this.viImageList.splice(index, 1)
        },
        // 预览图片
        handlePictureCardPreview(file) {
            this.dialogImageUrl = file.url || file
            this.dialogVisible = true
        }
    }
}
</script>

<style lang="scss" scoped>
.avatar-preview {
    object-fit: cover;
}
.el-upload__tip {
    font-size: 12px;
    color: #909399;
    margin-top: 5px;
}
</style>


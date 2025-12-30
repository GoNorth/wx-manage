<template>
    <el-dialog :title="!dataForm.salesId ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible" width="600px">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="100px">
            <el-form-item label="销售账号" prop="salesAccount">
                <el-input v-model="dataForm.salesAccount" placeholder="销售账号" :disabled="!!dataForm.salesId"></el-input>
            </el-form-item>
            <el-form-item label="销售姓名" prop="salesName">
                <el-input v-model="dataForm.salesName" placeholder="销售姓名"></el-input>
            </el-form-item>
            <el-form-item label="性别" prop="gender">
                <el-radio-group v-model="dataForm.gender">
                    <el-radio label="MALE">男</el-radio>
                    <el-radio label="FEMALE">女</el-radio>
                </el-radio-group>
            </el-form-item>
            <el-form-item label="年龄" prop="age">
                <el-input-number v-model="dataForm.age" :min="1" :max="150" placeholder="年龄" style="width: 100%;"></el-input-number>
            </el-form-item>
            <el-form-item label="手机号" prop="phone">
                <el-input v-model="dataForm.phone" placeholder="手机号"></el-input>
            </el-form-item>
            <el-form-item label="住址" prop="address">
                <el-input v-model="dataForm.address" placeholder="住址" type="textarea" :rows="3"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
        </span>
    </el-dialog>
</template>

<script>
export default {
    data() {
        return {
            visible: false,
            dataForm: {
                salesId: '',
                salesAccount: '',
                salesName: '',
                gender: 'MALE',
                age: null,
                phone: '',
                address: ''
            },
            dataRule: {
                salesAccount: [
                    { required: true, message: '销售账号不能为空', trigger: 'blur' },
                    { min: 3, max: 50, message: '销售账号长度在3到50个字符', trigger: 'blur' }
                ],
                salesName: [
                    { required: true, message: '销售姓名不能为空', trigger: 'blur' },
                    { min: 1, max: 50, message: '销售姓名长度在1到50个字符', trigger: 'blur' }
                ],
                gender: [
                    { required: true, message: '请选择性别', trigger: 'change' }
                ],
                phone: [
                    { required: true, message: '手机号不能为空', trigger: 'blur' },
                    { pattern: /^1[3-9]\d{9}$/, message: '请输入正确的手机号码', trigger: 'blur' }
                ],
                address: [
                    { max: 255, message: '住址长度不能超过255个字符', trigger: 'blur' }
                ]
            }
        }
    },
    methods: {
        init(item) {
            this.visible = true
            this.$nextTick(() => {
                this.$refs['dataForm'] && this.$refs['dataForm'].resetFields()
            })
            
            if (item && item.salesId) {
                // 修改模式，加载数据
                const wxOpenid = this.$cookie.get('wx_openid') || ''
                this.$http({
                    url: this.$http.adornUrl(`/manage/bizSales/info/${item.salesId}`),
                    method: 'get',
                    params: this.$http.adornParams(),
                    headers: {
                        'wx_openid': wxOpenid
                    }
                }).then(({ data }) => {
                    if (data && data.code === 200) {
                        Object.assign(this.dataForm, data.bizSales || {})
                    } else {
                        this.$message.error(data.msg || '获取数据失败')
                    }
                }).catch(() => {
                    // 如果接口不存在，使用传入的数据
                    Object.assign(this.dataForm, item)
                })
            } else {
                // 新增模式
                this.dataForm = {
                    salesId: '',
                    salesAccount: '',
                    salesName: '',
                    gender: 'MALE',
                    age: null,
                    phone: '',
                    address: ''
                }
            }
        },
        // 表单提交
        dataFormSubmit() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    const wxOpenid = this.$cookie.get('wx_openid') || ''
                    this.$http({
                        url: this.$http.adornUrl(`/manage/bizSales/${!this.dataForm.salesId ? 'save' : 'update'}`),
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
                }
            })
        }
    }
}
</script>


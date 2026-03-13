<template>
    <div class="app-container">
        <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch">
            <el-form-item label="医生姓名" prop="doctorName">
                <el-input v-model="queryParams.doctorName" placeholder="请输入医生姓名" clearable style="width: 240px"
                    @keyup.enter.native="handleQuery" />
            </el-form-item>
            <el-form-item label="科室" prop="department">
                <el-input v-model="queryParams.department" placeholder="请输入科室" clearable style="width: 240px"
                    @keyup.enter.native="handleQuery" />
            </el-form-item>
            <el-form-item label="联系电话" prop="phone">
                <el-input v-model="queryParams.phone" placeholder="请输入联系电话" clearable style="width: 240px"
                    @keyup.enter.native="handleQuery" />
            </el-form-item>
            <el-form-item label="性别" prop="gender">
                <el-select v-model="queryParams.gender" placeholder="请选择性别" clearable style="width: 240px">
                    <el-option v-for="dict in dict.type.sys_user_sex" :key="dict.value" :label="dict.label"
                        :value="dict.value" />
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
            </el-form-item>
        </el-form>

        <el-row :gutter="10" class="mb8">
            <el-col :span="1.5">
                <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleAdd">新增</el-button>
            </el-col>
            <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
        </el-row>

        <el-table v-loading="loading" :data="doctorList" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55" align="center" />
            <el-table-column label="医生姓名" prop="doctorName" :show-overflow-tooltip="true" width="150" />
            <el-table-column label="性别" prop="gender" :show-overflow-tooltip="true" width="120">
                <template slot-scope="scope">
                    <span v-if="scope.row.gender === '0'">男</span>
                    <span v-else-if="scope.row.gender === '1'">女</span>
                    <span v-else>{{ scope.row.gender }}</span>
                </template>
            </el-table-column>
            <el-table-column label="科室" prop="department" :show-overflow-tooltip="true" width="150" />
            <el-table-column label="联系电话" prop="phone" :show-overflow-tooltip="true" width="150" />
            <el-table-column label="职称" prop="title" :show-overflow-tooltip="true" width="120" />
            <el-table-column label="邮箱" prop="email" :show-overflow-tooltip="true" width="180" />
            <el-table-column label="入职时间" prop="hireDate" :show-overflow-tooltip="true" width="120" />
            <el-table-column label="操作" fixed="right" align="center" class-name="small-padding fixed-width" width="180">
                <template slot-scope="scope">
                    <el-button size="mini" type="text" icon="el-icon-view" @click="handleView(scope.row)"
                        v-hasPermi="['system:doctor:view']">查看</el-button>
                    <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)"
                        v-hasPermi="['system:doctor:edit']">修改</el-button>
                    <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)"
                        v-hasPermi="['system:doctor:remove']">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <pagination v-show="total > 0" :total="total" :page.sync="queryParams.pageNum"
            :limit.sync="queryParams.pageSize" @pagination="getList" />

        <!-- 添加或修改医生信息对话框 -->
        <el-dialog :title="title" :visible.sync="open" width="60%" append-to-body :fullscreen="false"
            :max-height="'80vh'" :close-on-click-modal="false">
            <el-form ref="form" :model="form" :rules="rules" label-width="120px">
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="医生姓名" prop="doctorName">
                            <el-input v-model="form.doctorName" placeholder="请输入医生姓名" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="性别" prop="gender">
                            <el-select v-model="form.gender" placeholder="请选择性别" style="width: 100%">
                                <el-option v-for="dict in dict.type.sys_user_sex" :key="dict.value" :label="dict.label"
                                    :value="dict.value" />
                            </el-select>
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="科室" prop="department">
                            <el-input v-model="form.department" placeholder="请输入科室" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="职称" prop="title">
                            <el-input v-model="form.title" placeholder="请输入职称" />
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="联系电话" prop="phone">
                            <el-input v-model="form.phone" placeholder="请输入联系电话" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="邮箱" prop="email">
                            <el-input v-model="form.email" placeholder="请输入邮箱" />
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="入职时间" prop="hireDate">
                            <el-date-picker v-model="form.hireDate" type="date" placeholder="选择入职时间" style="width: 100%"
                                value-format="yyyy-MM-dd" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="状态" prop="status">
                            <el-select v-model="form.status" placeholder="请选择状态" style="width: 100%">
                                <el-option label="在职" value="1" />
                                <el-option label="离职" value="0" />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="医生图片" prop="doctorImg">
                            <el-upload action="https://yiliao.admin.php7788.com/prod-api/system/file/upload" list-type="picture-card"
                                :file-list="doctorImgList" :on-preview="handlePictureCardPreview"
                                :on-success="handleDoctorImgSuccess" :on-remove="handleDoctorImgRemove"
                                :before-upload="beforeDoctorImgUpload" :limit="1" multiple
                                :data="{ bizType: 'doctor' }">
                                <i class="el-icon-plus"></i>
                                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，单个不超过500KB，最多1张</div>
                            </el-upload>
                            <el-dialog :visible.sync="dialogVisible" append-to-body>
                                <img width="100%" :src="dialogImageUrl" alt="">
                            </el-dialog>
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-form-item label="医生简介" prop="introduction">
                    <el-input v-model="form.introduction" :rows="4" type="textarea" placeholder="请输入医生简介" />
                </el-form-item>
                <el-form-item label="备注" prop="remark">
                    <el-input v-model="form.remark" :rows="3" type="textarea" placeholder="请输入备注" />
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitForm">确 定</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>

        <!-- 查看医生详情对话框 -->
        <el-dialog :title="'医生详情'" :visible.sync="viewVisible" width="800px" append-to-body>
            <el-form ref="viewForm" :model="viewForm" label-width="120px">
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="医生姓名">
                            <el-input v-model="viewForm.doctorName" disabled />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="性别">
                            <el-input v-model="viewForm.gender" disabled />
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="科室">
                            <el-input v-model="viewForm.department" disabled />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="职称">
                            <el-input v-model="viewForm.title" disabled />
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="联系电话">
                            <el-input v-model="viewForm.phone" disabled />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="邮箱">
                            <el-input v-model="viewForm.email" disabled />
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="入职时间">
                            <el-input v-model="viewForm.hireDate" disabled />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="状态">
                            <el-input v-model="viewForm.status" disabled />
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-col :span="24">
                    <el-form-item label="医生图片" prop="doctorImg">
                        <el-upload action="https://yiliao.admin.php7788.com/prod-api/system/file/upload" list-type="picture-card"
                            :file-list="doctorImgList" :on-preview="handlePictureCardPreview"
                            :on-success="handleDoctorImgSuccess" :on-remove="handleDoctorImgRemove"
                            :before-upload="beforeDoctorImgUpload" :limit="1" multiple :data="{ bizType: 'doctor' }">
                            <i class="el-icon-plus"></i>
                            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，单个不超过500KB，最多1张</div>
                        </el-upload>
                        <el-dialog :visible.sync="dialogVisible" append-to-body>
                            <img width="100%" :src="dialogImageUrl" alt="">
                        </el-dialog>
                    </el-form-item>
                </el-col>
                <el-form-item label="医生简介">
                    <el-input v-model="viewForm.introduction" :rows="4" type="textarea" disabled />
                </el-form-item>
                <el-form-item label="备注">
                    <el-input v-model="viewForm.remark" :rows="3" type="textarea" disabled />
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="viewCancel">关 闭</el-button>
            </div>
        </el-dialog>

    </div>
</template>

<script>
import request from "@/utils/request"

export default {
    name: "Doctor",
    dicts: ['sys_user_sex'],
    data() {
        return {
            // 遮罩层
            loading: true,
            // 选中数组
            ids: [],
            // 非单个禁用
            single: true,
            // 非多个禁用
            multiple: true,
            // 显示搜索条件
            showSearch: true,
            // 总条数
            total: 0,
            // 医生表格数据
            doctorList: [],
            // 弹出层标题
            title: "",
            // 是否显示弹出层
            open: false,
            // 是否显示详情弹出层
            viewVisible: false,
            // 日期范围
            dateRange: [],
            // 查询参数
            queryParams: {
                pageNum: 1,
                pageSize: 10,
                doctorName: undefined,
                department: undefined,
                phone: undefined,
                gender: undefined
            },
            // 表单参数
            form: {
                doctorId: undefined,
                doctorName: undefined,
                gender: undefined,
                department: undefined,
                title: undefined,
                phone: undefined,
                email: undefined,
                hireDate: undefined,
                status: undefined,
                introduction: undefined,
                remark: undefined
            },
            // 详情表单参数
            viewForm: {
                doctorId: undefined,
                doctorName: undefined,
                gender: undefined,
                department: undefined,
                title: undefined,
                phone: undefined,
                email: undefined,
                hireDate: undefined,
                status: undefined,
                introduction: undefined,
                remark: undefined
            },
            // 表单校验
            rules: {
                doctorName: [
                    { required: true, message: "医生姓名不能为空", trigger: "blur" }
                ],
                gender: [
                    { required: true, message: "性别不能为空", trigger: "change" }
                ],
                phone: [
                    { required: true, message: "联系电话不能为空", trigger: "blur" },
                    { pattern: /^1[3-9]\d{9}$/, message: "请输入有效的11位手机号码", trigger: "blur" }
                ],
                email: [
                    { type: "email", message: "请输入正确的邮箱格式", trigger: ["blur", "change"] }
                ],
                introduction: [
                    { required: true, message: "医生简介不能为空", trigger: "blur" }
                ]
            },
            // 医生图片列表
            doctorImgList: [],
            // 图片预览
            dialogImageUrl: '',
            dialogVisible: false
        }
    },
    created() {
        this.getList()
    },
    methods: {
        /** 查询医生列表 */
        getList() {
            this.loading = true
            request({
                url: '/system/doctor/list',
                method: 'get',
                params: this.addDateRange(this.queryParams, this.dateRange)
            }).then(response => {
                this.doctorList = response.rows
                this.total = response.total
                this.loading = false
            }
            )
        },
        // 取消按钮
        cancel() {
            this.open = false
            this.reset()
        },
        // 详情取消按钮
        viewCancel() {
            this.viewVisible = false
        },
        // 表单重置
        reset() {
            this.form = {
                doctorId: undefined,
                doctorName: undefined,
                gender: undefined,
                department: undefined,
                title: undefined,
                phone: undefined,
                email: undefined,
                hireDate: undefined,
                status: undefined,
                introduction: undefined,
                remark: undefined
            }
            this.resetForm("form")
        },
        /** 搜索按钮操作 */
        handleQuery() {
            this.queryParams.pageNum = 1
            this.getList()
        },
        /** 重置按钮操作 */
        resetQuery() {
            this.dateRange = []
            this.resetForm("queryForm")
            this.handleQuery()
        },
        // 多选框选中数据
        handleSelectionChange(selection) {
            this.ids = selection.map(item => item.doctorId)
            this.single = selection.length != 1
            this.multiple = !selection.length
        },
        /** 新增按钮操作 */
        handleAdd() {
            this.reset()
            this.open = true
            this.title = "添加医生"
        },
        /** 修改按钮操作 */
        handleUpdate(row) {
            this.reset()
            this.doctorImgList = []
            const doctorId = row.doctorId || this.ids
            request({
                url: '/system/doctor/' + doctorId,
                method: 'get'
            }).then(response => {
                this.form = response.data
                // 处理医生图片的回显
                if (this.form.doctorImg) {
                    const cleanUrlStr = this.form.doctorImg.trim().replace(/^`|`$/g, '')
                    if (cleanUrlStr) {
                        this.doctorImgList = cleanUrlStr.split(',').map(url => {
                            const cleanUrl = url.trim()
                            if (cleanUrl.startsWith('http://') || cleanUrl.startsWith('https://')) {
                                return { url: cleanUrl }
                            } else {
                                return { url: 'http://8.142.96.116:8080' + cleanUrl }
                            }
                        }).filter(item => item.url)
                    }
                }
                this.open = true
                this.title = "修改医生"
            })
        },
        /** 查看详情按钮操作 */
        handleView(row) {
            const doctorId = row.doctorId || this.ids
            request({
                url: '/system/doctor/' + doctorId,
                method: 'get'
            }).then(response => {
                this.viewForm = response.data
                // 处理医生图片的回显
                if (this.viewForm.doctorImg) {
                    const cleanUrlStr = this.viewForm.doctorImg.trim().replace(/^`|`$/g, '')
                    if (cleanUrlStr) {
                        this.doctorImgList = cleanUrlStr.split(',').map(url => {
                            const cleanUrl = url.trim()
                            if (cleanUrl.startsWith('http://') || cleanUrl.startsWith('https://')) {
                                return { url: cleanUrl }
                            } else {
                                return { url: 'https://yiliao.admin.php7788.com' + cleanUrl }
                            }
                        }).filter(item => item.url)
                    }
                }
                console.log(this.doctorImgList, 'this.doctorImgList===')
                this.viewVisible = true
            })
        },
        /** 删除按钮操作 */
        handleDelete(row) {
            const doctorIds = row.doctorId || this.ids
            this.$modal.confirm('是否确认删除医生编号为"' + doctorIds + '"的数据项？').then(function () {
                return request({
                    url: '/system/doctor/' + doctorIds,
                    method: 'delete'
                })
            }).then(() => {
                this.getList()
                this.$modal.msgSuccess("删除成功")
            }).catch(() => { })
        },
        /** 提交按钮 */
        submitForm: function () {
            this.$refs["form"].validate(valid => {
                if (valid) {
                    console.log(this.form, 'this.form===')
                    if (this.form.doctorId != undefined) {
                        // this.form.doctorImg = 'https://yiliao.admin.php7788.com' + this.form.doctorImg
                        request({
                            url: '/system/doctor',
                            method: 'put',
                            data: this.form
                        }).then(() => {
                            this.$modal.msgSuccess("修改成功")
                            this.open = false
                            this.getList()
                        })
                    } else {
                        this.form.doctorImg = 'https://yiliao.admin.php7788.com' + this.form.doctorImg
                        request({
                            url: '/system/doctor',
                            method: 'post',
                            data: this.form
                        }).then(() => {
                            this.$modal.msgSuccess("新增成功")
                            this.open = false
                            this.getList()
                        })
                    }
                }
            })
        },
        /** 医生图片上传成功 */
        handleDoctorImgSuccess(response, file, fileList) {
            if (response.code === 200) {
                this.doctorImgList = fileList
                // 确保保存的是服务器返回的URL，而不是blob URL
                this.form.doctorImg = response.data
                console.log(this.form.doctorImg, 'this.form.doctorImg===')
                this.$message.success('上传成功')
            } else {
                this.$message.error(response.msg || '上传失败')
            }
        },
        /** 医生图片移除 */
        handleDoctorImgRemove(file, fileList) {
            this.doctorImgList = fileList
            this.form.doctorImg = fileList.map(item => item.response.url).join(',')
        },
        /** 医生图片预览 */
        handlePictureCardPreview(file) {
            this.dialogImageUrl = file.url
            this.dialogVisible = true
        },
        /** 医生图片上传前校验 */
        beforeDoctorImgUpload(file) {
            const isJPG = file.type === 'image/jpeg'
            const isPNG = file.type === 'image/png'
            const isLt500K = file.size / 1024 < 500

            if (!isJPG && !isPNG) {
                this.$message.error('上传图片只能是 JPG 或 PNG 格式!')
                return false
            }
            if (!isLt500K) {
                this.$message.error('上传图片大小不能超过 500KB!')
                return false
            }
            return true
        },
        /** 导出按钮操作 */
        handleExport() {
            this.download('system/doctor/export', {
                ...this.queryParams
            }, `doctor_${new Date().getTime()}.xlsx`)
        }
    }
}
</script>

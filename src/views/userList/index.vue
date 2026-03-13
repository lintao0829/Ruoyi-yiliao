<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch">
      <el-form-item label="患者姓名" prop="patientName">
        <el-input v-model="queryParams.patientName" placeholder="请输入患者姓名" clearable style="width: 240px"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="联系电话" prop="phone">
        <el-input v-model="queryParams.phone" placeholder="请输入联系电话" clearable style="width: 240px"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="性别" prop="gender">
        <el-select v-model="queryParams.gender" placeholder="请选择性别" clearable style="width: 240px">
          <el-option v-for="dict in dict.type.sys_user_sex" :key="dict.value" :label="dict.label" :value="dict.value" />
        </el-select>
      </el-form-item>
      <el-form-item label="主治医生" prop="doctorName">
        <el-select v-model="queryParams.doctorName" placeholder="请选择主治医生" clearable style="width: 240px">
          <el-option v-for="doctor in doctorList" :key="doctor.value" :label="doctor.label" :value="doctor.label" />
        </el-select>
      </el-form-item>
      <el-form-item label="科室" prop="department">
        <el-input v-model="queryParams.department" placeholder="请输入科室" clearable style="width: 240px"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleAdd">新增病例</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="warning" plain icon="el-icon-download" size="mini" @click="handleExport"
          v-hasPermi="['system:role:export']">导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="patientList" @selection-change="handleSelectionChange">
      <el-table-column label="患者姓名" prop="patientName" :show-overflow-tooltip="true" width="120" align="center" />
      <el-table-column label="性别" prop="gender" :show-overflow-tooltip="true" width="80" align="center">
        <template slot-scope="scope">
          <span v-if="scope.row.gender === '0'">男</span>
          <span v-else-if="scope.row.gender === '1'">女</span>
          <span v-else>{{ scope.row.gender }}</span>
        </template>
      </el-table-column>
      <el-table-column label="年龄" prop="age" width="80" align="center" />
      <el-table-column label="联系电话" prop="phone" :show-overflow-tooltip="true" width="150" align="center" />
      <el-table-column label="地址" prop="address" :show-overflow-tooltip="true" width="200" align="center" />
      <el-table-column label="科室" prop="department" :show-overflow-tooltip="true" align="center" />
      <el-table-column label="主治医生" prop="doctorName" :show-overflow-tooltip="true" width="200" align="center" />
      <el-table-column label="就诊时间" prop="visitTime" :show-overflow-tooltip="true" width="180" align="center" />
      <el-table-column label="诊断结果" prop="diagnosisResult" :show-overflow-tooltip="true" align="center" width="150" />
      <el-table-column label="治疗建议" prop="treatmentSuggestion" :show-overflow-tooltip="true" align="center"
        width="160" />
      <el-table-column label="备注" prop="remark" :show-overflow-tooltip="true" align="center" />
      <el-table-column label="操作" fixed="right" align="center" class-name="small-padding fixed-width" width="230">
        <template slot-scope="scope">
          <el-button size="mini" type="text" icon="el-icon-view" @click="handleView(scope.row)">查看</el-button>
          <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)">修改</el-button>
          <!--  v-hasPermi="['system:patient:edit']" -->
          <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)">删除</el-button>
          <!-- v-hasPermi="['system:patient:remove']" -->
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total > 0" :total="total" :page.sync="queryParams.pageNum" :limit.sync="queryParams.pageSize"
      @pagination="getList" />

    <!-- 添加或修改患者信息对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="900px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="120px">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="患者姓名" prop="patientName">
              <el-input v-model="form.patientName" placeholder="请输入患者姓名" />
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
            <el-form-item label="年龄" prop="age">
              <el-input-number v-model="form.age" controls-position="right" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="联系电话" prop="phone">
              <el-input v-model="form.phone" placeholder="请输入联系电话" />
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
            <el-form-item label="主治医生" prop="doctorId">
              <el-select v-model="form.doctorId" placeholder="请选择主治医生" clearable style="width: 100%"
                @change="handleDoctorChange" multiple>
                <el-option v-for="doctor in doctorList" :key="doctor.value" :label="doctor.label"
                  :value="doctor.value" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="就诊时间" prop="visitTime">
              <el-date-picker v-model="form.visitTime" type="datetime" placeholder="选择就诊时间" style="width: 100%"
                value-format="yyyy-MM-dd HH:mm:ss" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="病历图片" prop="imgUrl">
              <el-upload action="https://yiliao.admin.php7788.com/prod-api/system/file/upload" list-type="picture-card"
                :file-list="medicalRecordImgList" :on-preview="handlePictureCardPreview" :on-success="handleMedicalImgSuccess"
                :on-remove="handleMedicalImgRemove" :before-upload="beforeMedicalImgUpload" :limit="5" multiple :data="{ bizType: 'record' }">
                <i class="el-icon-plus"></i>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，单个不超过500KB，最多5张</div>
              </el-upload>
              <el-dialog :visible.sync="dialogVisible" append-to-body>
                <img width="100%" :src="dialogImageUrl" alt="">
              </el-dialog>
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="地址" prop="address">
          <el-input v-model="form.address" type="textarea" placeholder="请输入地址" />
        </el-form-item>
        <el-form-item label="诊断结果" prop="diagnosisResult">
          <el-input v-model="form.diagnosisResult" :rows="4" type="textarea" placeholder="请输入诊断结果" />
        </el-form-item>
        <el-form-item label="治疗建议" prop="treatmentSuggestion">
          <el-input v-model="form.treatmentSuggestion" :rows="4" type="textarea" placeholder="请输入治疗建议" />
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

    <!-- 查看患者信息对话框 -->
    <el-dialog title="患者详情" :visible.sync="viewOpen" width="900px" append-to-body>
      <el-form ref="viewForm" :model="viewForm" label-width="120px" disabled>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="患者姓名" prop="patientName">
              <el-input v-model="viewForm.patientName" placeholder="请输入患者姓名" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="性别" prop="gender">
              <el-select v-model="viewForm.gender" placeholder="请选择性别" style="width: 100%">
                <el-option v-for="dict in dict.type.sys_user_sex" :key="dict.value" :label="dict.label"
                  :value="dict.value" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="年龄" prop="age">
              <el-input-number v-model="viewForm.age" controls-position="right" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="联系电话" prop="phone">
              <el-input v-model="viewForm.phone" placeholder="请输入联系电话" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="科室" prop="department">
              <el-input v-model="viewForm.department" placeholder="请输入科室" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="主治医生" prop="doctorId">
              <el-select v-model="viewForm.doctorId" placeholder="请选择主治医生" style="width: 100%" multiple>
                <el-option v-for="doctor in doctorList" :key="doctor.value" :label="doctor.label"
                  :value="doctor.value" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="就诊时间" prop="visitTime">
              <el-date-picker v-model="viewForm.visitTime" type="datetime" placeholder="请选择就诊时间" style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="病历图片" prop="imgUrl">
              <el-upload disabled action="/system/file/upload" list-type="picture-card"
                :file-list="medicalRecordImgList" :on-preview="handlePictureCardPreview"
                :data="{ bizType: 'record' }">
                <i class="el-icon-plus"></i>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，单个不超过500KB，最多5张</div>
              </el-upload>
              <el-dialog :visible.sync="dialogVisible" append-to-body>
                <img width="100%" :src="dialogImageUrl" alt="">
              </el-dialog>
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="地址" prop="address">
          <el-input v-model="viewForm.address" type="textarea" placeholder="请输入地址" />
        </el-form-item>
        <el-form-item label="诊断结果" prop="diagnosisResult">
          <el-input v-model="viewForm.diagnosisResult" :rows="4" type="textarea" placeholder="请输入诊断结果" />
        </el-form-item>
        <el-form-item label="病历备注" prop="medicalRemark">
          <el-input v-model="viewForm.medicalRemark" :rows="4" type="textarea" placeholder="请输入病历备注" />
        </el-form-item>
        <el-form-item label="治疗建议" prop="treatmentSuggestion">
          <el-input v-model="viewForm.treatmentSuggestion" :rows="4" type="textarea" placeholder="请输入治疗建议" />
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="viewForm.remark" :rows="3" type="textarea" placeholder="请输入备注" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="viewOpen = false">关 闭</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import { listPatient, getPatient, delPatient, addPatient, updatePatient } from "@/api/system/patient"
import request from "@/utils/request"

export default {
  name: "Patient",
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
      // 患者表格数据
      patientList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 日期范围
      dateRange: [],
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        patientName: undefined,
        phone: undefined,
        gender: undefined,
        doctorName: undefined,
        department: undefined
      },
      // 表单参数
      form: {
        patientId: undefined,
        patientName: undefined,
        gender: undefined,
        age: undefined,
        phone: undefined,
        address: undefined,
        department: undefined,
        diagnosisResult: undefined,
        doctorId: undefined,
        doctorName: undefined,
        imgUrl: undefined,
        medicalRemark: undefined,
        remark: undefined,
        treatmentSuggestion: undefined,
        visitTime: undefined
      },
      // 表单校验
      rules: {
        patientName: [
          { required: true, message: "患者姓名不能为空", trigger: "blur" }
        ],
        phone: [
          { required: true, message: "联系电话不能为空", trigger: "blur" }
        ],
        gender: [
          { required: true, message: "性别不能为空", trigger: "change" }
        ]
      },
      // 医生列表
      doctorList: [],
      // 病历图片列表
      medicalRecordImgList: [],
      // 图片预览
      dialogImageUrl: '',
      dialogVisible: false,
      // 查看弹窗
      viewOpen: false,
      viewForm: {}
    }
  },
  created() {
    this.getList()
    this.getDoctorList()
  },
  methods: {
    /** 查询患者列表 */
    getList() {
      this.loading = true
      listPatient(this.addDateRange(this.queryParams, this.dateRange)).then(response => {
        this.patientList = response.rows
        this.total = response.total
        this.loading = false
      }
      )
    },
    /** 获取医生列表 */
    getDoctorList() {
      request({
        url: '/system/user/querySysUserList',
        method: 'get'
      }).then(response => {
        if (response.code === 200) {
          // 实际返回的数据格式为 { code: 200, rows: [{ doctorId: 1, doctorName: '医生测试1' }] }
          this.doctorList = response.rows.map(item => ({
            value: item.userId,
            label: item.userName
          }))
        }
      }).catch(() => {
        // 接口调用失败时使用默认数据
        this.doctorList = [
        ]
      })
    },
    /** 查看患者信息 */
    handleView(row) {
      const patientId = row.patientId || this.ids
      getPatient(patientId).then(response => {
        this.viewForm = response.data
        // 处理医生ID的多选情况
        if (this.viewForm.doctorId && typeof this.viewForm.doctorId === 'string') {
          this.viewForm.doctorId = this.viewForm.doctorId.split(',').map(id => id.trim())
        }
        // 处理病历图片的回显
        if (this.viewForm.imgUrl) {
          // 清理字符串：去除首尾空格和反引号
          const cleanUrlStr = this.viewForm.imgUrl.trim().replace(/^`|`$/g, '')
          if (cleanUrlStr) {
            this.medicalRecordImgList = cleanUrlStr.split(',').map(url => {
              const cleanUrl = url.trim()
              // 如果URL已包含完整地址，直接使用；否则拼接基础地址
              if (cleanUrl.startsWith('http://') || cleanUrl.startsWith('https://')) {
                return { url: cleanUrl }
              } else {
                return { url: 'https://yiliao.admin.php7788.com' + cleanUrl }
              }
            }).filter(item => item.url) // 过滤空URL
          }
        }
        this.viewOpen = true
      })
    },
    // 取消按钮
    cancel() {
      this.open = false
      this.reset()
    },
    // 表单重置
    reset() {
      this.form = {
        patientId: undefined,
        patientName: undefined,
        gender: undefined,
        age: undefined,
        phone: undefined,
        address: undefined,
        department: undefined,
        diagnosisResult: undefined,
        doctorId: undefined,
        doctorName: undefined,
        imgUrl: undefined,
        medicalRemark: undefined,
        remark: undefined,
        treatmentSuggestion: undefined,
        visitTime: undefined
      }
      this.medicalRecordImgList = []
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
      this.ids = selection.map(item => item.patientId)
      this.single = selection.length != 1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset()
      this.open = true
      this.title = "添加患者"
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset()
      const patientId = row.patientId || this.ids
      getPatient(patientId).then(response => {
        this.form = response.data
        // 处理医生ID的多选情况，转换为数字类型匹配医生列表的value
        if (this.form.doctorId) {
          if (typeof this.form.doctorId === 'string') {
            this.form.doctorId = this.form.doctorId.split(',').map(id => Number(id.trim()))
          } else if (Array.isArray(this.form.doctorId)) {
            this.form.doctorId = this.form.doctorId.map(id => Number(id))
          }
        }
        // 处理医生姓名的回显
        if (this.form.doctorId) {
          const doctorIds = Array.isArray(this.form.doctorId) ? this.form.doctorId : [this.form.doctorId]
          const doctorNames = doctorIds.map(id => {
            const doctor = this.doctorList.find(doctor => doctor.value === id)
            return doctor ? doctor.label : id
          }).filter(name => name)
          this.form.doctorName = doctorNames.join(', ')
        }
        // 处理病历图片的回显
        if (this.form.imgUrl) {
          // 清理字符串：去除首尾空格和反引号
          const cleanUrlStr = this.form.imgUrl.trim().replace(/^`|`$/g, '')
          if (cleanUrlStr) {
            this.medicalRecordImgList = cleanUrlStr.split(',').map(url => {
              const cleanUrl = url.trim()
              // 如果URL已包含完整地址，直接使用；否则拼接基础地址
              if (cleanUrl.startsWith('http://') || cleanUrl.startsWith('https://')) {
                return { url: cleanUrl }
              } else {
                return { url: 'https://yiliao.admin.php7788.com' + cleanUrl }
              }
            }).filter(item => item.url) // 过滤空URL
          }
        }
        this.open = true
        this.title = "修改患者信息"
      })
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const patientIds = row.patientId || this.ids
      this.$modal.confirm('是否确认删除患者编号为"' + patientIds + '"的数据项？').then(function () {
        return delPatient(patientIds)
      }).then(() => {
        this.getList()
        this.$modal.msgSuccess("删除成功")
      }).catch(() => { })
    },
    /** 提交按钮 */
    submitForm: function () {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.patientId != undefined) {
            updatePatient(this.form).then(() => {
              this.$modal.msgSuccess("修改成功")
              this.open = false
              this.getList()
            })
          } else {
            addPatient(this.form).then(() => {
              this.$modal.msgSuccess("新增成功")
              this.open = false
              this.getList()
            })
          }
        }
      })
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('system/patient/export', {
        ...this.queryParams
      }, `patient_${new Date().getTime()}.xlsx`)
    },
    /** 医生选择改变 */
    handleDoctorChange(value) {
      if (Array.isArray(value)) {
        // 多选情况
        const doctorNames = value.map(item => {
          const doctor = this.doctorList.find(doctor => doctor.value == item)
          return doctor ? doctor.label : ''
        }).filter(name => name)
        this.form.doctorName = doctorNames.join(', ')
      } else {
        // 单选情况
        const doctor = this.doctorList.find(item => item.value == value)
        if (doctor) {
          this.form.doctorName = doctor.label
        }
      }
    },
    /** 图片上传前校验 */
    beforeMedicalImgUpload(file) {
      const isJPG = file.type === 'image/jpeg'
      const isPNG = file.type === 'image/png'
      const isLt500K = file.size / 1024 <= 500

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
    /** 图片上传成功 */
    handleMedicalImgSuccess(response, file, fileList) {
      console.log(response,fileList,'response====')
      if (response.code === 200) {
        this.medicalRecordImgList = fileList
        // 将图片URL以逗号分隔存储
        const urls = fileList.map(item => {
          if (item.response) {
            return item.response.data
          }
          return item.url
        }).join(',')
        this.form.imgUrl = urls
        console.log(this.form,'this.form====')
        this.$message.success('上传成功')
      } else {
        this.$message.error(response.msg || '上传失败')
      }
    },
    /** 病历图片移除 */
    handleMedicalImgRemove(file, fileList) {
      this.medicalRecordImgList = fileList
      const urls = fileList.map(item => {
        if (item.response) {
          return item.response.url
        }
        return item.url
      }).join(',')
      this.form.imgUrl = urls
    },
    /** 图片预览 */
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    }
  }
}
</script>

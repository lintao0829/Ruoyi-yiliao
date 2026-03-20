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
      <el-table-column label="医生备注" prop="remark" :show-overflow-tooltip="true" align="center" />
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
    <el-dialog :title="title" :visible.sync="open" width="900px" append-to-body :close-on-click-modal="false">
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
          <!-- <el-col :span="24">
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
          </el-col> -->
        </el-row>
        <el-form-item label="地址" prop="address">
          <el-input v-model="form.address" type="textarea" placeholder="请输入地址" />
        </el-form-item>
        <!-- <el-form-item label="诊断结果" prop="diagnosisResult">
          <el-input v-model="form.diagnosisResult" :rows="4" type="textarea" placeholder="请输入诊断结果" />
        </el-form-item>
        <el-form-item label="治疗建议" prop="treatmentSuggestion">
          <el-input v-model="form.treatmentSuggestion" :rows="4" type="textarea" placeholder="请输入治疗建议" />
        </el-form-item>
        <el-form-item label="医生备注" prop="remark">
          <el-input v-model="form.remark" :rows="3" type="textarea" placeholder="请输入医生备注" />
        </el-form-item> -->
      </el-form>
      <!-- 就诊记录列表 -->
      <div v-if="form.patientId" class="record-section"
        style="margin-top: 20px; border-top: 1px solid #eee; padding-top: 20px;">
        <div class="record-header"
          style="margin-bottom: 15px; display: flex; justify-content: space-between; align-items: center;">
          <h3 style="margin: 0; font-size: 16px; font-weight: bold;">就诊记录列表</h3>
          <el-button type="primary" size="small" icon="el-icon-plus" @click="handleAddRecord">新增记录</el-button>
        </div>
        <el-table v-loading="recordLoading" :data="recordList" border style="width: 100%">
          <el-table-column type="index" label="序号" width="50" align="center" />
          <el-table-column label="就诊时间" prop="createTime" width="160" align="center" />
          <el-table-column label="糖化血红蛋白(%)" prop="sugarValue" width="120" align="center" />
          <el-table-column label="收缩压(mmHg)" prop="systolicPressure" width="110" align="center" />
          <el-table-column label="舒张压(mmHg)" prop="diastolicPressure" width="110" align="center" />
          <el-table-column label="低密度脂蛋白(mmol/L)" prop="ldlValue" width="150" align="center" />
          <el-table-column label="身高(cm)" prop="height" width="90" align="center" />
          <el-table-column label="体重(kg)" prop="weight" width="90" align="center" />
          <el-table-column label="吸烟支数" prop="smokingCount" width="90" align="center" />
          <el-table-column label="总评分" prop="totalScore" width="80" align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.totalScore >= 10 ? 'danger' : scope.row.totalScore >= 5 ? 'warning' : 'success'">
                {{ scope.row.totalScore }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="风险等级" prop="riskLevel" width="100" align="center">
            <template slot-scope="scope">
              <el-tag
                :type="scope.row.riskLevel === '高危' ? 'danger' : scope.row.riskLevel === '中危' ? 'warning' : 'success'">
                {{ scope.row.riskLevel }}
              </el-tag>
            </template>
          </el-table-column>

          <el-table-column label="操作" align="center" width="200" fixed="right">
            <template slot-scope="scope">
              <el-button size="mini" type="text" icon="el-icon-view" @click="handleViewRecord(scope.row)">查看</el-button>
              <el-button size="mini" type="text" icon="el-icon-edit"
                @click="handleUpdateRecord(scope.row)">编辑</el-button>
              <el-button size="mini" type="text" icon="el-icon-delete"
                @click="handleDeleteRecord(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <pagination v-show="recordTotal > 0" :total="recordTotal" :page.sync="recordQueryParams.pageNum"
          :limit.sync="recordQueryParams.pageSize" @pagination="getRecordList" />
      </div>

      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <!-- 查看患者信息对话框 -->
    <el-dialog title="患者详情" :visible.sync="viewOpen" width="900px" append-to-body :close-on-click-modal="false">
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
               <el-select v-model="viewForm.doctorId" placeholder="请选择主治医生" clearable style="width: 100%"
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
              <el-date-picker v-model="viewForm.visitTime" type="datetime" placeholder="请选择就诊时间" style="width: 100%" />
            </el-form-item>
          </el-col>
          <!-- <el-col :span="24">
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
          </el-col> -->
        </el-row>
        <el-form-item label="地址" prop="address">
          <el-input v-model="viewForm.address" type="textarea" placeholder="请输入地址" />
        </el-form-item>
        <!-- <el-form-item label="诊断结果" prop="diagnosisResult">
          <el-input v-model="viewForm.diagnosisResult" :rows="4" type="textarea" placeholder="请输入诊断结果" />
        </el-form-item>
        <el-form-item label="病历医生备注" prop="medicalRemark">
          <el-input v-model="viewForm.medicalRemark" :rows="4" type="textarea" placeholder="请输入病历医生备注" />
        </el-form-item>
        <el-form-item label="治疗建议" prop="treatmentSuggestion">
          <el-input v-model="viewForm.treatmentSuggestion" :rows="4" type="textarea" placeholder="请输入治疗建议" />
        </el-form-item> -->
        <!-- <el-form-item label="医生备注" prop="remark">
          <el-input v-model="viewForm.remark" :rows="3" type="textarea" placeholder="请输入医生备注" />
        </el-form-item> -->
        <el-table v-loading="recordLoading" :data="recordList" border style="width: 100%">
          <el-table-column type="index" label="序号" width="50" align="center" />
          <el-table-column label="就诊时间" prop="createTime" width="160" align="center" />
          <el-table-column label="糖化血红蛋白(%)" prop="sugarValue" width="120" align="center" />
          <el-table-column label="收缩压(mmHg)" prop="systolicPressure" width="110" align="center" />
          <el-table-column label="舒张压(mmHg)" prop="diastolicPressure" width="110" align="center" />
          <el-table-column label="低密度脂蛋白(mmol/L)" prop="ldlValue" width="150" align="center" />
          <el-table-column label="身高(cm)" prop="height" width="90" align="center" />
          <el-table-column label="体重(kg)" prop="weight" width="90" align="center" />
          <el-table-column label="吸烟支数" prop="smokingCount" width="90" align="center" />
          <el-table-column label="总评分" prop="totalScore" width="80" align="center">
            <template slot-scope="scope">
              <el-tag :type="scope.row.totalScore >= 10 ? 'danger' : scope.row.totalScore >= 5 ? 'warning' : 'success'">
                {{ scope.row.totalScore }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="风险等级" prop="riskLevel" width="100" align="center">
            <template slot-scope="scope">
              <el-tag
                :type="scope.row.riskLevel === '高危' ? 'danger' : scope.row.riskLevel === '中危' ? 'warning' : 'success'">
                {{ scope.row.riskLevel }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" align="center" width="200" fixed="right">
            <template slot-scope="scope">
              <el-button size="mini" type="text" icon="el-icon-view" :disabled="false"
                @click="handleViewRecord(scope.row)">查看</el-button>
              <el-button size="mini" type="text" icon="el-icon-edit"
                @click="handleUpdateRecord(scope.row)">编辑</el-button>
              <el-button size="mini" type="text" icon="el-icon-delete"
                @click="handleDeleteRecord(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="viewOpen = false">关 闭</el-button>
      </div>
    </el-dialog>

    <!-- 就诊记录弹窗 -->
    <el-dialog :title="recordTitle" :visible.sync="recordOpen" width="700px" append-to-body
      :close-on-click-modal="false">
      <el-form ref="recordForm" :model="recordForm" :rules="recordRules" label-width="140px"
        :disabled="!isRecordEdit && recordTitle === '查看就诊记录'">
        <!-- 评分结果展示（仅查看模式） -->
        <div style=" padding: 30px; background-color: #f5f7fa; margin-bottom: 20px; border-radius: 4px;">
          <h4 style="margin-top: 0; margin-bottom: 10px;">评分结果</h4>
          <el-row :gutter="20">
            <el-col :span="8">
              <div style="text-align: center;">
                <div style="font-size: 24px; font-weight: bold; color: #409EFF;">{{ recordForm.sugarScore }}</div>
                <div style="font-size: 12px; color: #909399;">糖化得分</div>
              </div>
            </el-col>
            <el-col :span="8">
              <div style="text-align: center;">
                <div style="font-size: 24px; font-weight: bold; color: #409EFF;">{{ recordForm.pressureScore }}</div>
                <div style="font-size: 12px; color: #909399;">血压得分</div>
              </div>
            </el-col>
            <el-col :span="8">
              <div style="text-align: center;">
                <div style="font-size: 24px; font-weight: bold; color: #409EFF;">{{ recordForm.ldlScore }}</div>
                <div style="font-size: 12px; color: #909399;">血脂得分</div>
              </div>
            </el-col>
          </el-row>
          <el-row :gutter="20" style="margin-top: 15px;">
            <el-col :span="8">
              <div style="text-align: center;">
                <div style="font-size: 24px; font-weight: bold; color: #409EFF;">{{ recordForm.bmiScore }}</div>
                <div style="font-size: 12px; color: #909399;">BMI得分</div>
              </div>
            </el-col>
            <el-col :span="8">
              <div style="text-align: center;">
                <div style="font-size: 24px; font-weight: bold; color: #409EFF;">{{ recordForm.smokingScore }}</div>
                <div style="font-size: 12px; color: #909399;">吸烟得分</div>
              </div>
            </el-col>
            <el-col :span="8">
              <div style="text-align: center;">
                <div style="font-size: 24px; font-weight: bold;"
                  :style="{ color: recordForm.totalScore >= 10 ? '#F56C6C' : recordForm.totalScore >= 5 ? '#E6A23C' : '#67C23A' }">
                  {{ recordForm.totalScore }}</div>
                <div style="font-size: 12px; color: #909399;">总评分</div>
              </div>
            </el-col>
          </el-row>
          <div style="margin-top: 15px; text-align: center;">
            <el-tag
              :type="recordForm.riskLevel === '高危' ? 'danger' : recordForm.riskLevel === '中危' ? 'warning' : 'success'"
              size="medium">
              风险等级：{{ recordForm.riskLevel }}
            </el-tag>
          </div>
        </div>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="糖化血红蛋白(%)" prop="sugarValue">
              <el-input-number v-model="recordForm.sugarValue" :precision="1" :step="0.1" :min="0"
                style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="收缩压(mmHg)" prop="systolicPressure">
              <el-input-number v-model="recordForm.systolicPressure" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="舒张压(mmHg)" prop="diastolicPressure">
              <el-input-number v-model="recordForm.diastolicPressure" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="低密度脂蛋白(mmol/L)" prop="ldlValue">
              <el-input-number v-model="recordForm.ldlValue" :precision="2" :step="0.01" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="身高(cm)" prop="height">
              <el-input-number v-model="recordForm.height" :precision="1" :step="0.1" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="体重(kg)" prop="weight">
              <el-input-number v-model="recordForm.weight" :precision="1" :step="0.1" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="吸烟支数" prop="smokingCount">
              <el-input-number v-model="recordForm.smokingCount" :min="0" style="width: 100%" />
            </el-form-item>
          </el-col>
        </el-row>
        <!-- 病历图片 -->
        <el-row :gutter="20">
          <el-col :span="24">
            <el-form-item label="病历图片" prop="imgUrl">
              <el-upload :action="uploadUrl" list-type="picture-card" :file-list="recordImgList"
                :on-preview="handlePictureCardPreview" :on-success="handleRecordImgSuccess"
                :on-remove="handleRecordImgRemove" :before-upload="beforeMedicalImgUpload" :limit="5" multiple
                :data="{ bizType: 'record' }" :disabled="!isRecordEdit && recordTitle === '查看就诊记录'">
                <i class="el-icon-plus"></i>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，单个不超过500KB，最多5张</div>
              </el-upload>
              <el-dialog :visible.sync="dialogVisible" append-to-body>
                <img width="100%" :src="dialogImageUrl" alt="">
              </el-dialog>
            </el-form-item>
          </el-col>
        </el-row>
        <!-- 诊断结果 -->
        <el-form-item label="诊断结果" prop="diagnosisResult">
          <el-input v-model="recordForm.diagnosisResult" :rows="4" type="textarea" placeholder="请输入诊断结果"
            :disabled="!isRecordEdit && recordTitle === '查看就诊记录'" />
        </el-form-item>
        <!-- 治疗建议 -->
        <el-form-item label="治疗建议" prop="treatmentSuggestion">
          <el-input v-model="recordForm.treatmentSuggestion" :rows="4" type="textarea" placeholder="请输入治疗建议"
            :disabled="!isRecordEdit && recordTitle === '查看就诊记录'" />
        </el-form-item>
        <!-- 医生备注 -->
        <el-form-item label="医生备注" prop="remark">
          <el-input v-model="recordForm.remark" :rows="3" type="textarea" placeholder="请输入医生备注"
            :disabled="!isRecordEdit && recordTitle === '查看就诊记录'" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button v-if="isRecordEdit || recordTitle === '新增就诊记录'" type="primary" @click="submitRecordForm">确
          定</el-button>
        <el-button @click="cancelRecord">取 消</el-button>
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
        // gender: [
        //   { required: true, message: "性别不能为空", trigger: "change" }
        // ]
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
      viewForm: {},
      // 就诊记录相关
      recordLoading: false,
      recordList: [],
      recordTotal: 0,
      recordQueryParams: {
        pageNum: 1,
        pageSize: 10
      },
      recordOpen: false,
      recordTitle: "",
      recordForm: {},
      isRecordEdit: false,
      recordImgList: [],
      uploadUrl: "https://yiliao.admin.php7788.com/prod-api/system/file/upload",
      // 就诊记录表单校验
      recordRules: {
        sugarValue: [
          { required: true, message: "糖化血红蛋白不能为空", trigger: "blur" }
        ],
        systolicPressure: [
          { required: true, message: "收缩压不能为空", trigger: "blur" }
        ],
        diastolicPressure: [
          { required: true, message: "舒张压不能为空", trigger: "blur" }
        ],
        ldlValue: [
          { required: true, message: "低密度脂蛋白不能为空", trigger: "blur" }
        ],
        height: [
          { required: true, message: "身高不能为空", trigger: "blur" }
        ],
        weight: [
          { required: true, message: "体重不能为空", trigger: "blur" }
        ],
        smokingCount: [
          { required: true, message: "吸烟支数不能为空", trigger: "blur" }
        ]
      }
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
    /** 查询就诊记录列表 */
    getRecordList() {
      console.log(this.form.patientId, 'this.form.patientId=======')
      if (!this.form.patientId) return
      this.recordLoading = true
      request({
        url: `https://yiliao.admin.php7788.com/prod-api/system/score/record/list?patientId=${this.form.patientId}`,
        method: 'get'
      }).then(response => {
        if (response.code === 200) {
          this.recordList = response.rows
          this.recordTotal = response.total
        }
        this.recordLoading = false
      }).catch(() => {
        this.recordLoading = false
      })
    },
    /** 新增就诊记录 */
    handleAddRecord() {
      this.recordForm = {
        patientId: this.form.patientId,
        sugarValue: undefined,
        systolicPressure: undefined,
        diastolicPressure: undefined,
        ldlValue: undefined,
        height: undefined,
        weight: undefined,
        smokingCount: undefined,
        diagnosisResult: undefined,
        treatmentSuggestion: undefined,
        remark: undefined,
        imgUrl: undefined
      }
      this.recordImgList = []
      this.isRecordEdit = false
      this.recordTitle = "新增就诊记录"
      this.recordOpen = true
    },
    /** 查看就诊记录 */
    handleViewRecord(row) {
      this.recordForm = { ...row }
      this.isRecordEdit = false
      this.recordTitle = "查看就诊记录"
      // 处理病历图片的回显
      this.recordImgList = []
      // this.recordImgList.push({ url: 'https://yiliao.admin.php7788.com' + row.imgUrl })
      if (row.imgUrl) {
        const cleanUrlStr = row.imgUrl.trim().replace(/^`|`$/g, '')
        if (cleanUrlStr) {
          this.recordImgList = cleanUrlStr.split(',').map(url => {
            const cleanUrl = url.trim()
            if (cleanUrl.startsWith('http://') || cleanUrl.startsWith('https://')) {
              return { url: cleanUrl }
            } else {
              return { url: 'https://yiliao.admin.php7788.com' + cleanUrl }
            }
          }).filter(item => item.url)
        }
      }
      this.recordOpen = true
    },
    /** 编辑就诊记录 */
    handleUpdateRecord(row) {
      this.recordForm = { ...row }
      this.isRecordEdit = true
      this.recordTitle = "编辑就诊记录"
      // 处理病历图片的回显
      this.recordImgList = []
      if (row.imgUrl) {
        const cleanUrlStr = row.imgUrl.trim().replace(/^`|`$/g, '')
        if (cleanUrlStr) {
          this.recordImgList = cleanUrlStr.split(',').map(url => {
            const cleanUrl = url.trim()
            if (cleanUrl.startsWith('http://') || cleanUrl.startsWith('https://')) {
              return { url: cleanUrl }
            } else {
              return { url: 'https://yiliao.admin.php7788.com' + cleanUrl }
            }
          }).filter(item => item.url)
        }
      }
      this.recordOpen = true
    },
    /** 删除就诊记录 */
    handleDeleteRecord(row) {
      this.$modal.confirm('是否确认删除该就诊记录？').then(() => {
        return request({
          url: `/system/score/record/${row.recordId}`,
          method: 'delete'
        })
      }).then(() => {
        this.getRecordList()
        this.$modal.msgSuccess("删除成功")
      }).catch(() => { })
    },
    /** 提交就诊记录 */
    submitRecordForm() {
      console.log(this.recordForm, 'this.recordForm=====')
      this.$refs["recordForm"].validate(valid => {
        if (valid) {
          const url = this.isRecordEdit ? '/system/score/record' : '/system/score/record/submit'
          const method = this.isRecordEdit ? 'put' : 'post'
          request({
            url: url,
            method: method,
            data: this.recordForm
          }).then(() => {
            this.$modal.msgSuccess(this.isRecordEdit ? "修改成功" : "新增成功")
            this.recordOpen = false
            this.getRecordList()
          })
        }
      })
    },
    /** 取消就诊记录 */
    cancelRecord() {
      this.recordOpen = false
      this.recordImgList = []
    },
    /** 病历图片上传成功 */
    handleRecordImgSuccess(response, file, fileList) {
      if (response.code === 200) {
        this.recordImgList = fileList
        // 将所有图片URL以逗号分隔存储，去掉域名前缀
        const urls = fileList.map(item => {
          if (item.response) {
            let url = item.response.data
            // 去掉https://yiliao.admin.php7788.com前缀
            if (url.startsWith('https://yiliao.admin.php7788.com')) {
              url = url.replace('https://yiliao.admin.php7788.com', '')
            }
            return url
          }
          return item.url
        }).join(',')
        this.recordForm.imgUrl = urls
        this.$message.success('上传成功')
      } else {
        this.$message.error(response.msg || '上传失败')
      }
    },
    /** 病历图片移除 */
    handleRecordImgRemove(file, fileList) {
      this.recordImgList = fileList
      const urls = fileList.map(item => {
        if (item.response) {
          let url = item.response.data
          // 去掉https://yiliao.admin.php7788.com前缀
          if (url.startsWith('https://yiliao.admin.php7788.com')) {
            url = url.replace('https://yiliao.admin.php7788.com', '')
          }
          return url
        }
        return item.url
      }).join(',')
      this.recordForm.imgUrl = urls
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
      this.form.patientId = row.patientId
      const patientId = row.patientId || this.ids
      getPatient(patientId).then(response => {
        this.viewForm = response.data
        // 处理医生ID的多选情况，转换为数字类型匹配医生列表的value
        if (this.viewForm.doctorId) {
          if (typeof this.viewForm.doctorId === 'string') {
            this.viewForm.doctorId = this.viewForm.doctorId.split(',').map(id => Number(id.trim()))
          } else if (Array.isArray(this.viewForm.doctorId)) {
            this.viewForm.doctorId = this.viewForm.doctorId.map(id => Number(id))
          }
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
        this.getRecordList()
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
        // 加载就诊记录列表
        this.$nextTick(() => {
          this.getRecordList()
        })
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
      console.log(response, fileList, 'response====')
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
        console.log(this.form, 'this.form====')
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

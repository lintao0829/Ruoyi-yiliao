<template>
  <div class="clinic-home">
    <!-- 顶部导航栏 -->
    <div class="top-nav">
      <div class="nav-left">
        <div class="logo">
          <i class="el-icon-medal"></i>
          <span>医疗管理系统</span>
        </div>
      </div>
      <div class="nav-right">
        <div class="user-info">
          <i class="el-icon-user"></i>
          <span>{{ doctorName }}</span>
        </div>
      </div>
    </div>

    <!-- 欢迎区域 -->
    <div class="welcome-section">
      <div class="welcome-content">
        <h1 class="welcome-title">欢迎登录，{{ doctorName }}医生</h1>
        <p class="welcome-subtitle">今天是 {{ currentDate }}，祝您工作顺利</p>
      </div>
      <div class="weather-widget">
        <div class="weather-icon">
          <i class="el-icon-sunny"></i>
        </div>
        <div class="weather-info">
          <div class="weather-temp">22°C</div>
          <div class="weather-desc">晴天</div>
        </div>
        <div class="weather-details">
          <div class="weather-detail-item">
            <span class="detail-label">湿度</span>
            <span class="detail-value">45%</span>
          </div>
          <div class="weather-detail-item">
            <span class="detail-label">风力</span>
            <span class="detail-value">3级</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 统计卡片区域 -->
    <div class="stats-section">
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-icon bg-blue">
            <i class="el-icon-user"></i>
          </div>
          <div class="stat-content">
            <div class="stat-number">28</div>
            <div class="stat-label">今日接诊</div>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon bg-green">
            <i class="el-icon-time"></i>
          </div>
          <div class="stat-content">
            <div class="stat-number">5</div>
            <div class="stat-label">候诊人数</div>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon bg-orange">
            <i class="el-icon-check"></i>
          </div>
          <div class="stat-content">
            <div class="stat-number">23</div>
            <div class="stat-label">已完成</div>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon bg-purple">
            <i class="el-icon-document"></i>
          </div>
          <div class="stat-content">
            <div class="stat-number">15</div>
            <div class="stat-label">新增病历</div>
          </div>
        </div>
      </div>
    </div>

    <!-- 快捷操作区 -->
    <div class="actions-section">
      <h2 class="section-title">快捷操作</h2>
      <div class="actions-grid">
        <div class="action-card" @click="handleAction('/user_list')">
          <div class="action-icon bg-blue">
            <i class="el-icon-document"></i>
          </div>
          <div class="action-title">病历管理</div>
          <div class="action-desc">查看和管理患者病历</div>
        </div>
        <!-- <div class="action-card" @click="handleAction('/appointment')">
          <div class="action-icon bg-green">
            <i class="el-icon-date"></i>
          </div>
          <div class="action-title">预约管理</div>
          <div class="action-desc">查看和管理预约信息</div>
        </div> -->
        <!-- <div class="action-card" @click="handleAction('/prescription')">
          <div class="action-icon bg-orange">
            <i class="el-icon-medical"></i>
          </div>
          <div class="action-title">开具处方</div>
          <div class="action-desc">为患者开具电子处方</div>
        </div> -->
        <!-- <div class="action-card" @click="handleAction('/statistics')">
          <div class="action-icon bg-purple">
            <i class="el-icon-data-analysis"></i>
          </div>
          <div class="action-title">统计报表</div>
          <div class="action-desc">查看诊所运营数据</div>
        </div> -->
      </div>
    </div>

    <!-- 最近患者区域 -->
    <div class="recent-patients-section">
      <h2 class="section-title">最近就诊患者</h2>
      <div class="patients-table">
        <table>
          <thead>
            <tr>
              <th>姓名</th>
              <th>性别</th>
              <th>年龄</th>
              <th>就诊时间</th>
              <th>诊断结果</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="patient in recentPatients" :key="patient.id">
              <td>{{ patient.name }}</td>
              <td>{{ patient.gender }}</td>
              <td>{{ patient.age }}</td>
              <td>{{ patient.visitTime }}</td>
              <td>{{ patient.diagnosis }}</td>
              <td>
                <el-button type="primary" size="mini" @click="viewPatient(patient.id)">
                  查看
                </el-button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- 底部信息 -->
    <div class="footer">
      <p>© 2026 康泰诊所管理系统 | 版本 1.0.0</p>
    </div>
  </div>
</template>

<script>
import Cookies from 'js-cookie'

export default {
  name: 'ClinicHome',
  data() {
    return {
      doctorName: Cookies.get('username'),
      currentDate: '',
      recentPatients: [
        { id: 1, name: '王小明', gender: '男', age: 32, visitTime: '2026-03-06 09:30', diagnosis: '感冒' },
        { id: 2, name: '李秀英', gender: '女', age: 45, visitTime: '2026-03-06 10:15', diagnosis: '高血压' },
        { id: 3, name: '陈建国', gender: '男', age: 58, visitTime: '2026-03-06 11:00', diagnosis: '糖尿病' },
        { id: 4, name: '刘美华', gender: '女', age: 29, visitTime: '2026-03-06 14:00', diagnosis: '过敏' },
        { id: 5, name: '赵志强', gender: '男', age: 41, visitTime: '2026-03-06 15:30', diagnosis: '胃炎' }
      ]
    }
  },
  created() {
    console.log(Cookies.get('username'),'登录用户')
    this.initDate()
  },
  methods: {
    initDate() {
      const now = new Date()
      const year = now.getFullYear()
      const month = now.getMonth() + 1
      const day = now.getDate()
      const weekDays = ['日', '一', '二', '三', '四', '五', '六']
      const weekDay = weekDays[now.getDay()]
      this.currentDate = `${year}年${month}月${day}日 星期${weekDay}`
    },
    handleAction(path) {
      this.$router.push(path)
    },
    viewPatient(patientId) {
      this.$router.push(`/user_list`)
    }
  }
}
</script>

<style lang="scss" scoped>
.clinic-home {
  min-height: 100vh;
  background-color: #f5f7fa;
  display: flex;
  flex-direction: column;
}

/* 顶部导航栏 */
.top-nav {
  background: white;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  padding: 0 30px;
  height: 60px;
  display: flex;
  justify-content: space-between;
  align-items: center;

  .logo {
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 18px;
    font-weight: 600;
    color: #667eea;

    i {
      font-size: 24px;
    }
  }

  .user-info {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 14px;
    color: #606266;

    i {
      font-size: 16px;
    }
  }
}

/* 欢迎区域 */
.welcome-section {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 40px 30px;
  color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;

  .welcome-title {
    font-size: 28px;
    font-weight: 600;
    margin-bottom: 10px;
  }

  .welcome-subtitle {
    font-size: 16px;
    opacity: 0.9;
  }
}

/* 天气小组件 */
.weather-widget {
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 20px;
  min-width: 250px;

  .weather-icon {
    font-size: 48px;
    color: #fff;
    filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.1));
  }

  .weather-info {
    flex: 1;

    .weather-temp {
      font-size: 28px;
      font-weight: 700;
      color: #fff;
      margin-bottom: 4px;
    }

    .weather-desc {
      font-size: 14px;
      color: rgba(255, 255, 255, 0.9);
    }
  }

  .weather-details {
    display: flex;
    flex-direction: column;
    gap: 8px;

    .weather-detail-item {
      display: flex;
      flex-direction: column;
      align-items: flex-end;

      .detail-label {
        font-size: 12px;
        color: rgba(255, 255, 255, 0.8);
        margin-bottom: 2px;
      }

      .detail-value {
        font-size: 14px;
        font-weight: 600;
        color: #fff;
      }
    }
  }
}

/* 统计卡片区域 */
.stats-section {
  padding: 30px;

  .stats-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;

    .stat-card {
      background: white;
      border-radius: 12px;
      padding: 24px;
      display: flex;
      align-items: center;
      gap: 20px;
      box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
      transition: all 0.3s ease;

      &:hover {
        transform: translateY(-4px);
        box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
      }

      .stat-icon {
        width: 60px;
        height: 60px;
        border-radius: 12px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 24px;
        color: white;
      }

      .stat-content {
        flex: 1;

        .stat-number {
          font-size: 28px;
          font-weight: 700;
          color: #303133;
          margin-bottom: 4px;
        }

        .stat-label {
          font-size: 14px;
          color: #909399;
        }
      }
    }
  }
}

/* 快捷操作区 */
.actions-section {
  padding: 0 30px 30px;

  .section-title {
    font-size: 20px;
    font-weight: 600;
    color: #303133;
    margin-bottom: 20px;
  }

  .actions-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;

    .action-card {
      background: white;
      border-radius: 12px;
      padding: 30px;
      text-align: center;
      cursor: pointer;
      transition: all 0.3s ease;
      box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);

      &:hover {
        transform: translateY(-4px);
        box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
      }

      .action-icon {
        width: 80px;
        height: 80px;
        border-radius: 20px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 32px;
        color: white;
        margin: 0 auto 20px;
      }

      .action-title {
        font-size: 16px;
        font-weight: 600;
        color: #303133;
        margin-bottom: 8px;
      }

      .action-desc {
        font-size: 14px;
        color: #909399;
      }
    }
  }
}

/* 最近患者区域 */
.recent-patients-section {
  padding: 0 30px 30px;

  .section-title {
    font-size: 20px;
    font-weight: 600;
    color: #303133;
    margin-bottom: 20px;
  }

  .patients-table {
    background: white;
    border-radius: 12px;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
    overflow: hidden;

    table {
      width: 100%;
      border-collapse: collapse;

      th, td {
        padding: 16px;
        text-align: left;
        border-bottom: 1px solid #ebeef5;
      }

      th {
        background-color: #f5f7fa;
        font-weight: 600;
        color: #303133;
      }

      tr:last-child td {
        border-bottom: none;
      }

      tr:hover {
        background-color: #f5f7fa;
      }
    }
  }
}

/* 底部信息 */
.footer {
  background: white;
  padding: 20px;
  text-align: center;
  border-top: 1px solid #ebeef5;
  margin-top: auto;

  p {
    font-size: 14px;
    color: #909399;
    margin: 0;
  }
}

/* 颜色类 */
.bg-blue {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.bg-green {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
}

.bg-orange {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.bg-purple {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .stats-grid,
  .actions-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .welcome-section {
    flex-direction: column;
    text-align: center;
    gap: 30px;
  }

  .weather-widget {
    width: 100%;
    justify-content: center;
  }

  .stats-grid,
  .actions-grid {
    grid-template-columns: 1fr;
  }

  .top-nav,
  .stats-section,
  .actions-section,
  .recent-patients-section {
    padding: 20px;
  }
}
</style>
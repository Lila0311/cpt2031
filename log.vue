<template>
  <div class="logs-container">
    <!-- 统计卡片 -->
    <div class="stats-cards">
      <el-card class="stat-card">
        <div class="stat-icon">
          <el-icon class="icon"><Reading /></el-icon>
        </div>
        <div class="stat-content">
          <div class="stat-value count-up">{{ totalBooks }}</div>
          <div class="stat-label">总书数</div>
        </div>
      </el-card>
      <el-card class="stat-card">
        <div class="stat-icon">
          <el-icon class="icon"><Clock /></el-icon>
        </div>
        <div class="stat-content">
          <div class="stat-value count-up">{{ totalMinutes }}</div>
          <div class="stat-label">总时长（分钟）</div>
        </div>
      </el-card>
      <el-card class="stat-card">
        <div class="stat-icon">
          <el-icon class="icon"><Timer /></el-icon>
        </div>
        <div class="stat-content">
          <div class="stat-value count-up">{{ averageMinutes }}</div>
          <div class="stat-label">平均时长（分钟）</div>
        </div>
      </el-card>
    </div>

    <el-card>
      <template #header>
        <div class="card-header">
          <span>阅读记录</span>
          <el-button type="primary" @click="handleAdd">新增记录</el-button>
        </div>
      </template>
      
      <el-table :data="logs" style="width: 100%">
        <el-table-column prop="bookTitle" label="书籍名称" />
        <el-table-column prop="readingMinutes" label="阅读时长(分钟)" />
        <el-table-column prop="readingDate" label="阅读日期" />
        <el-table-column prop="notes" label="笔记" />
        <el-table-column label="操作" width="200">
          <template #default="scope">
            <el-button size="small" @click="handleEdit(scope.row)">编辑</el-button>
            <el-button size="small" type="danger" @click="handleDelete(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      
      <!-- 当没有数据时，显示添加按钮 -->
      <div v-if="logs.length === 0" class="empty-data">
        <el-empty description="暂无阅读记录">
          <el-button type="primary" @click="handleAdd">添加记录</el-button>
        </el-empty>
      </div>
    </el-card>

    <!-- 图表区域 -->
    <div class="chart-container">
      <el-card class="chart-card">
        <template #header>
          <div class="card-header">
            <span>每日阅读时长</span>
          </div>
        </template>
        <div id="lineChart" class="chart"></div>
      </el-card>
      
      <el-card class="chart-card">
        <template #header>
          <div class="card-header">
            <span>阅读时长分布</span>
          </div>
        </template>
        <div id="pieChart" class="chart"></div>
      </el-card>
    </div>

    <!-- 新增/编辑对话框 -->
    <el-dialog
      :title="dialogTitle"
      v-model="dialogVisible"
      width="50%"
    >
      <el-form :model="logForm" label-width="100px">
        <el-form-item label="书籍名称">
          <el-input v-model="logForm.bookTitle"></el-input>
        </el-form-item>
        <el-form-item label="阅读时长">
          <el-input-number v-model="logForm.readingMinutes" :min="1"></el-input-number>
        </el-form-item>
        <el-form-item label="阅读日期">
          <el-date-picker
            v-model="logForm.readingDate"
            type="date"
            placeholder="选择阅读日期"
          />
        </el-form-item>
        <el-form-item label="笔记">
          <el-input
            type="textarea"
            v-model="logForm.notes"
            :rows="4"
            placeholder="请输入阅读笔记"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handleSubmit">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'
import * as echarts from 'echarts';
import { Reading, Clock, Timer } from '@element-plus/icons-vue';

export default {
  name: 'Logs',
  components: {
    Reading,
    Clock,
    Timer
  },
  data() {
    return {
      dialogVisible: false,
      dialogTitle: '新增记录',
      logForm: {
        bookTitle: '',
        readingMinutes: 30,
        readingDate: '',
        notes: ''
      },
      currentLogId: null,
      lineChart: null,
      pieChart: null,
      // 模拟数据
      mockLogs: [
        { id: 1, bookTitle: '三体', readingMinutes: 60, readingDate: '2025-04-01', notes: '刘慈欣的科幻巨著' },
        { id: 2, bookTitle: '活着', readingMinutes: 45, readingDate: '2025-04-03', notes: '余华的经典作品' },
        { id: 3, bookTitle: '三体', readingMinutes: 30, readingDate: '2025-04-05', notes: '继续阅读第二章' },
        { id: 4, bookTitle: '百年孤独', readingMinutes: 90, readingDate: '2025-04-07', notes: '魔幻现实主义代表作' },
        { id: 5, bookTitle: '活着', readingMinutes: 50, readingDate: '2025-04-09', notes: '读到了感人的章节' }
      ]
    }
  },
  computed: {
    ...mapState({
      // 从 store 获取数据，如果没有则使用模拟数据
      storeLogs: state => state.logs || []
    }),
    // 使用模拟数据或 store 数据
    logs() {
      return this.storeLogs.length > 0 ? this.storeLogs : this.mockLogs;
    },
    totalBooks() {
      // 计算不重复的书籍数量
      return this.logs && this.logs.length > 0 
        ? [...new Set(this.logs.map(log => log.bookTitle))].length 
        : 0;
    },
    totalMinutes() {
      // 计算总时长
      return this.logs && this.logs.length > 0 
        ? this.logs.reduce((sum, log) => sum + (log.readingMinutes || 0), 0) 
        : 0;
    },
    averageMinutes() {
      // 计算平均时长
      return this.logs && this.logs.length > 0 
        ? Math.round(this.totalMinutes / this.logs.length) 
        : 0;
    }
  },
  methods: {
    ...mapActions(['fetchLog', 'updateLog', 'deleteLog']),
    handleAdd() {
      this.dialogTitle = '新增记录'
      this.logForm = {
        bookTitle: '',
        readingMinutes: 30,
        readingDate: new Date(),
        notes: ''
      }
      this.currentLogId = null
      this.dialogVisible = true
    },
    async handleEdit(log) {
      this.dialogTitle = '编辑记录'
      this.logForm = { ...log }
      this.currentLogId = log.id
      this.dialogVisible = true
    },
    async handleDelete(log) {
      try {
        await this.$confirm('确认删除该记录吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        
        // 如果使用的是模拟数据，直接从本地删除
        if (this.storeLogs.length === 0) {
          const index = this.mockLogs.findIndex(item => item.id === log.id);
          if (index !== -1) {
            this.mockLogs.splice(index, 1);
          }
        } else {
          // 否则调用 store action
          await this.deleteLog(log.id);
        }
        
        this.$message.success('删除成功');
        this.updateCharts(); // 删除后更新图表
      } catch (error) {
        if (error !== 'cancel') {
          this.$message.error('删除失败');
        }
      }
    },
    async handleSubmit() {
      try {
        const formattedDate = this.formatDate(this.logForm.readingDate);
        const updatedLogForm = {
          ...this.logForm,
          readingDate: formattedDate
        };
        
        if (this.currentLogId) {
          // 如果使用的是模拟数据，直接更新本地数据
          if (this.storeLogs.length === 0) {
            const index = this.mockLogs.findIndex(item => item.id === this.currentLogId);
            if (index !== -1) {
              this.mockLogs[index] = { ...updatedLogForm, id: this.currentLogId };
            }
          } else {
            // 否则调用 store action
            await this.updateLog({
              logId: this.currentLogId,
              data: updatedLogForm
            });
          }
        } else {
          // 新增记录
          if (this.storeLogs.length === 0) {
            // 为模拟数据生成新的 ID
            const newId = this.mockLogs.length > 0 
              ? Math.max(...this.mockLogs.map(item => item.id)) + 1 
              : 1;
            this.mockLogs.push({ ...updatedLogForm, id: newId });
          } else {
            // 通过 store 创建记录
            // 实现 createLog action 调用
          }
        }
        
        this.dialogVisible = false;
        this.$message.success('保存成功');
        this.updateCharts(); // 保存后更新图表
        this.animateCountUp(); // 触发统计数字动画
      } catch (error) {
        this.$message.error('保存失败');
      }
    },
    
    formatDate(date) {
      if (!date) return '';
      if (typeof date === 'string') return date;
      
      const d = new Date(date);
      const year = d.getFullYear();
      const month = String(d.getMonth() + 1).padStart(2, '0');
      const day = String(d.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    },
    
    // 初始化图表
    initCharts() {
      // 初始化折线图
      this.lineChart = echarts.init(document.getElementById('lineChart'));
      
      // 初始化饼图
      this.pieChart = echarts.init(document.getElementById('pieChart'));
      
      // 更新图表
      this.updateCharts();
      
      // 添加窗口大小变化的监听器
      window.addEventListener('resize', this.resizeHandler);
    },
    
    // 处理窗口大小变化
    resizeHandler() {
      if (this.lineChart) {
        this.lineChart.resize();
      }
      if (this.pieChart) {
        this.pieChart.resize();
      }
    },
    
    // 更新图表
    updateCharts() {
      this.updateLineChart();
      this.updatePieChart();
    },
    
    // 更新折线图
    updateLineChart() {
      if (!this.lineChart || !this.logs || this.logs.length === 0) return;
      
      // 准备数据，按日期排序
      const sortedLogs = [...this.logs].sort((a, b) => 
        new Date(a.readingDate) - new Date(b.readingDate)
      );
      
      const dates = sortedLogs.map(log => log.readingDate);
      const minutes = sortedLogs.map(log => log.readingMinutes);
      
      // 设置折线图选项
      const option = {
        tooltip: {
          trigger: 'axis'
        },
        xAxis: {
          type: 'category',
          data: dates,
          axisLabel: {
            rotate: 45,
            interval: 0
          }
        },
        yAxis: {
          type: 'value',
          name: '阅读时长（分钟）'
        },
        series: [{
          data: minutes,
          type: 'line',
          smooth: true,
          lineStyle: {
            width: 3,
            color: '#409EFF'
          },
          itemStyle: {
            color: '#409EFF'
          },
          areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: 'rgba(64,158,255,0.7)' },
              { offset: 1, color: 'rgba(64,158,255,0.1)' }
            ])
          },
          // 添加动画效果
          animationDelay: function (idx) {
            return idx * 100;
          },
          animationDuration: 1000,
          animationEasing: 'cubicOut'
        }]
      };
      
      this.lineChart.setOption(option);
    },
    
    // 更新饼图
    updatePieChart() {
      if (!this.pieChart || !this.logs || this.logs.length === 0) return;
      
      // 按书籍名称聚合数据
      const bookData = {};
      this.logs.forEach(log => {
        if (bookData[log.bookTitle]) {
          bookData[log.bookTitle] += log.readingMinutes;
        } else {
          bookData[log.bookTitle] = log.readingMinutes;
        }
      });
      
      // 转换为饼图所需的数据格式
      const pieData = Object.keys(bookData).map(title => ({
        name: title,
        value: bookData[title]
      }));
      
      // 设置饼图选项
      const option = {
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} 分钟 ({d}%)'
        },
        legend: {
          orient: 'vertical',
          right: 10,
          top: 'center',
          data: Object.keys(bookData)
        },
        series: [{
          name: '阅读时长',
          type: 'pie',
          radius: ['50%', '70%'],
          avoidLabelOverlap: false,
          itemStyle: {
            borderRadius: 10,
            borderColor: '#fff',
            borderWidth: 2
          },
          label: {
            show: false,
            position: 'center'
          },
          emphasis: {
            label: {
              show: true,
              fontSize: '18',
              fontWeight: 'bold'
            },
            scaleSize: 10
          },
          labelLine: {
            show: false
          },
          data: pieData,
          // 添加动画效果
          animationType: 'scale',
          animationEasing: 'elasticOut',
          animationDelay: function (idx) {
            return idx * 100;
          },
          animationDuration: 1000
        }]
      };
      
      this.pieChart.setOption(option);
    },
    
    // 添加数字增长动画效果
    animateCountUp() {
      const elements = document.querySelectorAll('.count-up');
      elements.forEach(el => {
        // 添加动画类，然后移除以便重新触发
        el.classList.add('animate');
        setTimeout(() => {
          el.classList.remove('animate');
        }, 1000);
      });
    }
  },
  mounted() {
    // 尝试获取日志数据
    this.fetchLog && this.fetchLog();
    
    // 在组件挂载后初始化图表
    this.$nextTick(() => {
      this.initCharts();
      // 初始触发一次数字动画
      setTimeout(() => {
        this.animateCountUp();
      }, 500);
    });
  },
  beforeUnmount() {
    // 在组件销毁前清理图表实例和事件监听器
    if (this.lineChart) {
      this.lineChart.dispose();
    }
    if (this.pieChart) {
      this.pieChart.dispose();
    }
    window.removeEventListener('resize', this.resizeHandler);
  }
}
</script>

<style scoped>
.logs-container {
  padding: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stats-cards {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.stat-card {
  flex: 1;
  margin: 0 10px;
  text-align: center;
  background-color: #f5f7fa;
  border-radius: 8px;
  overflow: hidden;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  padding: 15px;
}

.stat-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
}

.stat-card:first-child {
  margin-left: 0;
}

.stat-card:last-child {
  margin-right: 0;
}

.stat-icon {
  background: linear-gradient(135deg, #67C23A, #409EFF);
  color: white;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 15px;
}

.stat-card:nth-child(2) .stat-icon {
  background: linear-gradient(135deg, #E6A23C, #F56C6C);
}

.stat-card:nth-child(3) .stat-icon {
  background: linear-gradient(135deg, #909399, #409EFF);
}

.icon {
  font-size: 24px;
}

.stat-content {
  flex: 1;
  text-align: left;
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
  color: #409EFF;
  transition: all 0.3s ease;
  position: relative;
}

.stat-card:nth-child(2) .stat-value {
  color: #E6A23C;
}

.stat-card:nth-child(3) .stat-value {
  color: #909399;
}

.stat-label {
  margin-top: 8px;
  color: #606266;
}

.chart-container {
  display: flex;
  margin-top: 20px;
  flex-wrap: wrap;
}

.chart-card {
  flex: 1;
  min-width: 45%;
  margin: 10px;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.chart-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
}

.chart {
  height: 300px;
  width: 100%;
}

.empty-data {
  padding: 40px 0;
  text-align: center;
}

/* 数字动画效果 */
.count-up.animate {
  animation: countUp 1s ease-out forwards;
}

@keyframes countUp {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
    color: #F56C6C;
  }
  100% {
    transform: scale(1);
  }
}

@media (max-width: 768px) {
  .stats-cards {
    flex-direction: column;
  }
  
  .stat-card {
    margin: 5px 0;
  }
  
  .chart-container {
    flex-direction: column;
  }
  
  .chart-card {
    width: 100%;
    margin: 10px 0;
  }
}
</style>

<template>
  <div class="user-tasks">
    <div class="tasks-header">
      <h3>消息中心</h3>
    </div>
    <div class="tasks-body">
      <el-scrollbar>
        <div v-if="tasks.length === 0" class="no-tasks">
          暂无新消息
        </div>
        <div v-else>
          <div 
            v-for="task in tasks" 
            :key="task.id" 
            class="task-item"
            :class="{ 'is-read': task.read }"
            @click="handleTaskClick(task)"
          >
            <div class="task-content">
              <div class="task-title">{{ task.name }}</div>
              <div class="task-details">
                <span>{{ task.area }} - {{ task.type }}</span>
              </div>
            </div>
            <div class="task-time">{{ formatTime(task.createdAt) }}</div>
          </div>
        </div>
      </el-scrollbar>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import request from '/@/utils/request';
import { useUserInfo } from '/@/stores/userInfo';
import { storeToRefs } from 'pinia';
import dayjs from 'dayjs';
import mittBus from '/@/utils/mitt';

const stores = useUserInfo();
const { userInfos } = storeToRefs(stores);
const tasks = ref<any[]>([]);
const hasUnread = ref(false);

const fetchTasks = async () => {
  if (userInfos.value && userInfos.value.userName) {
    try {
      const res = await request({
        url: '/api/detection-tasks',
        method: 'get',
        params: { username: userInfos.value.userName },
      });
      tasks.value = res.data;
      checkUnreadStatus();
    } catch (error) {
      console.error('Failed to fetch tasks:', error);
    }
  }
};

const handleTaskClick = async (task: any) => {
  if (!task.read) {
    try {
      await request({
        url: `/api/detection-tasks/${task.id}/read`,
        method: 'put',
      });
      task.read = true;
      checkUnreadStatus();
    } catch (error) {
      console.error('Failed to mark task as read:', error);
    }
  }
};

const checkUnreadStatus = () => {
  hasUnread.value = tasks.value.some(task => !task.read);
};

watch(hasUnread, (newValue) => {
  mittBus.emit('setUnreadStatus', newValue);
});

const formatTime = (time: string) => {
  return dayjs(time).format('YYYY-MM-DD HH:mm');
};

onMounted(() => {
  fetchTasks();
});
</script>

<style scoped lang="scss">
.user-tasks {
  display: flex;
  flex-direction: column;
  width: 300px;
}

.tasks-header {
  padding: 10px 15px;
  border-bottom: 1px solid #ebeef5;
  h3 {
    margin: 0;
    font-size: 16px;
  }
}

.tasks-body {
  height: 400px;
  .no-tasks {
    text-align: center;
    color: #909399;
    padding: 20px;
  }
}

.task-item {
  display: flex;
  justify-content: space-between;
  padding: 10px 15px;
  border-bottom: 1px solid #ebeef5;
  cursor: pointer;
  transition: background-color 0.3s;

  &.is-read {
    color: #909399;
    .task-title {
      font-weight: normal;
    }
  }

  &:hover {
    background-color: #f5f7fa;
  }

  &:last-child {
    border-bottom: none;
  }
}

.task-content {
  flex-grow: 1;
}

.task-title {
  font-weight: bold;
}

.task-details {
  font-size: 12px;
}

.task-time {
  font-size: 12px;
  flex-shrink: 0;
  margin-left: 10px;
}
</style> 
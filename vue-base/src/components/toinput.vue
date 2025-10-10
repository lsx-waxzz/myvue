<template>
  <!-- 待办事项应用的主容器 -->
  <div class="todo-container">
    <!-- 输入区域容器 -->
    <div class="input-container">
      <input 
        type="text" 
        v-model="todo" 
        placeholder="在这里写下你的待办事项..." 
        class="todo-input"
        @keyup.enter="addTodo"
      >
    </div>

    <!-- 待办列表容器 -->
    <div class="list-container">
      <div 
        v-for="(item, index) in displayedList"
        :key="index" 
        class="todo-item"
      >
        <!-- 自定义复选框 -->
        <label class="custom-checkbox">
          <input type="checkbox" 
                 v-model="item.finish"
                 @change="saveToLocalStorage"
          >
          <span class="checkmark"></span>
        </label>
        
        <!-- 
          待办文本：
          - :class="{ 'todo-completed': item.finish }"：根据完成状态动态添加划线样式
        -->
        <span :class="{ 'todo-completed': item.finish }">{{ item.text }}</span>
        
        <!-- 删除按钮-->
        <button class="delete-btn" @click="deleteTodo(index)">×</button>
      </div>
      
      <!-- 当列表为空时显示提示文本 -->
      <div v-if="displayedList.length === 0" class="empty-list">暂无待办事项</div>
    </div>

    <!-- 控制面板：显示剩余数量和视图切换按钮 -->
    <div class="control-panel">
      <!-- 显示剩余未完成的待办事项数量 -->
      <span>{{ remaining }}</span>
      
      <!-- 视图切换按钮组 -->
      <button @click="switchView = 'all'">全部</button>
      <button @click="switchView = 'active'">积极</button>
      <button @click="switchView = 'completed'">完成</button>
      
      <!-- 清除已完成的待办事项 -->
      <button @click="clearCompleted">清除完成</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      todo: '',
      todoList: [],
      switchView: 'all'
    }
  },
  computed: {
    displayedList() {
      if (this.switchView === 'all') {
        // 显示所有待办项
        return this.todoList;
      } else if (this.switchView === 'active') {
        // 只显示未完成的待办项（finish为false）
        return this.todoList.filter(item => !item.finish);
      } else if (this.switchView === 'completed') {
        // 只显示已完成的待办项（finish为true）
        return this.todoList.filter(item => item.finish);
      }
    },
    remaining() {
      return this.todoList.filter(item => !item.finish).length;
    }
  },
  methods: {
    addTodo() {
      // 去除输入文本的前后空格
      const text = this.todo.trim();
      if (!text){
          return;
      }
      // 检查是否存在相同内容的待办项，若存在则提示并返回
      if (this.todoList.some(item => item.text === text)) {
          alert('该待办事项已存在');
          return;
      }
      // 添加新待办项到列表，初始状态为未完成
      this.todoList.push({
          text,
          finish: false
      });
      // 清空输入框
      this.todo = '';
      // 保存到本地存储
      this.saveToLocalStorage();
    },
    deleteTodo(index) {
      // 从数组中删除对应索引的项（删除1个）
      this.todoList.splice(index, 1);
      // 保存到本地存储
      this.saveToLocalStorage();
    },
    clearCompleted() {
      this.todoList = this.todoList.filter(item => !item.finish);
      this.saveToLocalStorage();
    },
    saveToLocalStorage() {
      localStorage.setItem('todoList', JSON.stringify(this.todoList));
      localStorage.setItem('switchView', this.switchView);
    },
    loadFromLocalStorage() {
      const savedTodoList = localStorage.getItem('todoList');
      const savedView = localStorage.getItem('switchView');
      if (savedTodoList) {
        this.todoList = JSON.parse(savedTodoList);
      }
      if (savedView) {
        this.switchView = savedView;
      }
    }
  },
  mounted() {
    this.loadFromLocalStorage();
  }
}
</script>

<style scoped>
body{
    background-color: #eee; /* 页面背景色 */
    margin: 0;
    padding: 20px;
}

/* 待办应用主容器样式 */
.todo-container {
    max-width: 500px; 
    margin: 0 auto; 
    background-color: #fff; 
    padding: 20px;
    border-radius: 8px; 
    box-shadow: 0 2px 8px rgba(0,0,0,0.1); 
}

/* 输入区域容器样式 */
.input-container {
    margin-bottom: 20px; 
}

/* 输入框样式 */
.todo-input {
  width: 100%; 
  padding: 12px; 
  font-size: 16px; 
  border: none; 
  border-bottom: 1px solid #ddd; 
  outline: none; 
  box-sizing: border-box; 
}

/* 自定义复选框样式 */
.custom-checkbox {
  display: inline-block;
  position: relative; 
  padding-left: 30px; 
  cursor: pointer; 
  user-select: none; 
}

/* 隐藏原生复选框 */
.custom-checkbox input {
  position: absolute;
  opacity: 0; 
  cursor: pointer;
}

/* 自定义复选框的外观 */
.checkmark {
  position: absolute;
  top: 50%; 
  left: 0;
  height: 20px;
  width: 20px;
  transform: translateY(-50%); 
  border: 1px solid #ccc; 
  border-radius: 50%; 
}

/* 复选框选中状态的样式 */
.custom-checkbox input:checked ~ .checkmark {
  background-color: #4CAF50; 
}

/* 复选框选中后的对勾符号 */
.checkmark:after {
  content: "";
  position: absolute;
  display: none; 
}

/* 选中状态时显示对勾 */
.custom-checkbox input:checked ~ .checkmark:after {
  display: block;
}

/* 对勾的具体样式 */
.custom-checkbox .checkmark:after {
  left: 7px;
  top: 3px;
  width: 5px;
  height: 10px;
  border: solid white; 
  border-width: 0 2px 2px 0; 
  transform: rotate(45deg); 
}

/* 列表容器样式 */
.list-container {
  margin-top: 20px; 
}

/* 单个待办项样式 */
.todo-item {
  display: flex; 
  align-items: center; 
  padding: 20px 0; 
  border-bottom: 1px solid #ddd; 
}

/* 已完成待办项的文本样式 */
.todo-completed {
  text-decoration: line-through; 
  color: #939393; 
}

/* 删除按钮样式 */
.delete-btn {
  color: #f44336; 
  background: none; 
  border: none; 
  font-size: 20px; 
  cursor: pointer; 
  margin-left: auto; 
}

/* 空列表提示文本样式 */
.empty-list {
  text-align: center; 
  color: #999; 
  padding: 20px 0; 
}

/* 控制面板样式 */
.control-panel {
  display: flex; 
  justify-content: space-between; 
  align-items: center; 
  margin-top: 20px; 
}

/* 控制面板按钮样式 */
.control-panel button {
  padding: 6px 12px; 
  margin: 0 5px; 
  background-color: #fff; 
  border: 1px solid #ddd; 
  border-radius: 4px; 
  cursor: pointer; 
}
</style>
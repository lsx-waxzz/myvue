<template>
    <div class="header">
        <h1>婴儿名字流行度流图</h1>
        <div class="file-upload">
            <h3>选择文件</h3>
            <p>请选择CSV格式的文件</p>
            <div class="file-input-container">
                <input type="file" class="file-input" @change="handleFileUpload" :disabled="loading">
                <label class="file-input-button">
                    <span v-if="!loading">选择CSV文件</span>
                    <span v-else>正在上传...</span>
                </label>
            </div>
            <div class="file-name">已选择文件：{{filename}}</div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            data: [],
            filename: '',
            loading: false
        }
    },
    methods: {
        handleFileUpload(event) {
            const file = event.target.files[0];
            if(!file.name.toLowerCase().endsWith('.csv')){
                alert('请选择CSV文件！');
                return;
            }
            this.filename = file.name;
            this.loading = true;
            const reader = new FileReader();
            reader.onload = (e) => {
                const text = e.target.result;
                const parsedData = this.parseCSV(text);
                this.data = parsedData;
                this.loading = false;
                this.$emit('update:data', parsedData);
            }
            reader.readAsText(file);
        },
        parseCSV(text) {
            const lines = text.split('\n');
            const headers = lines[0].split(',');
            const data = [];
            for (let i = 1; i < lines.length; i++) {
                const value = lines[i].split(',').map(value => value.trim());
                if(value.length === 1 && value[0] === '') continue;
                const row = {};
                headers.forEach((header, j) => {
                    if(header === 'year'){
                        const year = parseInt(value[j]);
                        if(isNaN(year)) return;
                        row[header] = year;
                    }else{
                        const num = parseInt(value[j]);
                        if(isNaN(num)) return;
                        row[header] = num;
                    }
                });
                data.push(row);
            }
            console.log(data);
            return data.sort((a, b) => a.year - b.year);
        }
    }
}
</script>

<style scoped>
.header {
    margin-top: 0 auto;
    max-width: 1200px;
    padding: 20px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}
h1 {
  text-align: center;
  color: #2c3e50;
  margin: 0 auto;
  font-weight: 600;
}
.file-upload {
  text-align: center;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 8px;
  border: 2px dashed #dee2e6;
}
.file-input-container {
  position: relative;
  display: inline-block;
  margin: 10px 0;
}
.file-input {
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}
.file-input-button {
  display: inline-block;
  padding: 12px 24px;
  background: #4e79a7;
  color: white;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 500;
}
.file-input-button:hover {
  background: #3a5f8a;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.file-name {
  margin-top: 10px;
  font-size: 14px;
  color: #6c757d;
}
</style>
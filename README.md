  # yk-knowledge
  知识库组件
  
  # install
  1、从gitHub获取：在需要安装的文件中，右键选择git bash here，在弹出窗体中：
        输入命令：git clone -b master git@github.com:13544580197/yk-knowledge.git
 
  #组件参数说明
  
  
 #例子 在需要引入的vue文件中
 <template>
 <div class="app-container">
	<List ></List>
 </div>
 </template>
 
 <script>
	 import List from '@/components/yk-knowledge/src/components/Knowledge/list'
	 export default {
	   components: {
		 List
		}
	 }
 </script>
 
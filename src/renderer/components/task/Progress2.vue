<template>
        <el-main class="maincontent">
        
				<el-row class="myEl-Row">
					<p class="el-rowText">任务目标</p>
				</el-row>
				<p class="task-goal">{{taskGoal}}</p>
				<div class="MissionGoal"></div>
				<el-row class="myEl-Row">
					<p class="el-rowText">里程碑进度</p>
				</el-row>
				<div class="task-progress">
				<div class="hint">
					<span class="schedule">
						<i class="el-icon-success" style="color:#E41541;"></i>
						<span>已通过</span>
					</span>

					<span class="schedule">
						<i class="el-icon-warning" style="color:#000000;"></i>
						<span>待审核</span>
					</span>

					<span class="schedule">
						<i class="el-icon-time" style="color:#707070;"></i>
						<span>未开始</span>
					</span>
				</div>

				<div style="height:25%;margin:20px 0 60px;">
					<el-steps :active="completion" align-center>
						<el-step v-for="item in task" style="cursor:pointer" :id=item.id :title=item.title :description=item.des :icon=item.ic :titleName=item.title @click.native='handleClick($event)'></el-step>
						<!--
						<el-step style="cursor:pointer" titleName='详细设计' title="详细设计" description="2018/4/19" icon="el-icon-success" @click.native='handleClick($event)'></el-step>
						<el-step style="cursor:pointer" titleName='界面设计' title="界面设计" description="2018/4/20" icon="el-icon-warning" @click.native='handleClick($event)'></el-step>
						<el-step style="cursor:pointer" titleName='数据库设计' title="数据库设计" description="2018/4/25" icon="el-icon-time" @click.native='handleClick($event)'></el-step>//-->
					</el-steps>
				</div>

				<el-row class="myEl-Row">
					<p class="el-rowText">审核情况</p>
				</el-row>

				<div style="height:40px">
					<span style="font-size:20px;font-weight:bold">{{name}}</span>
					<span style="font-size:18px;font-weight:bold;color:#E41541">{{status}}</span>
				</div>

				<div v-if="status==='未通过'">
					<span style="font-size:20px;font-weight:bold">原因：</span>
					<span style="font-size:13px;font-weight:bold">{{reason}}</span>
				</div>
			</div>
			</el-main>
</template>

<script>
export default {
  mounted() {
    this.init()
  },
  name: 'Task-page',
  data () {
    return {
      activeName: 'first',
      completion: '',
      taskGoal: '',
      task:[],
      name:'',
      status:'',
      reason:''
    }
  },
  methods: {
  	init() {
  		let vm = this
        //console.log(vm.$route.params.taskId)
        //console.log(localStorage["taskId"])
        this.$http.get(          
          HOST + '/tasks/' + localStorage["taskId"], 
          {headers: {'Content-Type': 'application/json;charset=utf-8'}}
          ).then(response=>{
            //console.log(123)
            //console.log(response.data)
            vm.taskGoal = response.data.info
            vm.completion = response.data.completion
            //that.$router.replace('/facerecognition')
          }).catch(error=>{
            console.log(error);
        });
        this.$http.get(          
          HOST + '/tasks/' + localStorage["taskId"] + '/milestones', 
          {headers: {'Content-Type': 'application/json;charset=utf-8'}}
          ).then(response=>{
            //console.log(123)
            console.log(response.data)
            for(var i=0;i<response.data.length;i++)
            {
            	var ico;
            	if (response.data[i].status=="PASS")
            		ico = "el-icon-success"
            	else if (response.data[i].status=="NOTPASS")
            		ico = 'el-icon-warning'
            	else if(response.data[i].status=="NOTBEGIN")
            		ico = 'el-icon-time'
            	//console.log(ico)
            	this.task.push({
            		id:response.data[i].id,
            		title:response.data[i].name,
            		des:response.data[i].endTime.slice(0,10),
            		ic:ico
            	})
            }
           //vm.taskGoal = response.data.info
            //vm.completion = response.data.completion
            //that.$router.replace('/facerecognition')
          }).catch(error=>{
            console.log(error);
        });
        this.$http.get(          
          HOST + '/tasks/' + localStorage["taskId"] + '/milestoneHistory', 
          {headers: {'Content-Type': 'application/json;charset=utf-8'}}
          ).then(response=>{
            //console.log(123)
            //console.log(response.data)
            vm.name = response.data.milestone.name
            var sta;
              if (response.data.status == -1)
                sta = '未通过'
              else if (response.data.status == 0)
                sta = '已提交'
              else if (response.data.status == 1)
                sta = '已通过'
            vm.status = sta
            vm.reason = response.data.reason
            //that.$router.replace('/facerecognition')
          }).catch(error=>{
            console.log(error);
        });  
    },
    handleClick (event) {
    	//var a = event.currentTarget.getAttribute('id')
      localStorage.setItem('milestoneId',event.currentTarget.getAttribute('id'))
     // console.log(a)
      this.$router.push({path: '/contractee/homePage/task/detail/milestone'})
      this.$emit('changeThirdBread', event.currentTarget.getAttribute('titleName'))
    } 
  }
}
</script>

<style>
.task-progress{
	margin-left: 0px;
}
.hint{
	margin-bottom: 20px;
}
/*任务目标下的一块空间*/
.MissionGoal {
  width:100%;
  height:20px;
}

/*里程碑进度下面的提示符*/
.schedule {
  width:12%;
  height:10%;
}
</style>
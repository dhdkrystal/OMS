<template>
	<el-main class="maincontent">

      <el-row class="myEl-Row">
        <font class="el-rowText">里程碑目标</font>
      </el-row>
      <p>{{info}}</p>
      <div style="height:10px"></div>

      <el-row class="myEl-Row">
        <font class="el-rowText">截止日期</font>
      </el-row>
      <p>{{endtime}}</p>
      <div style="height:10px"></div>

      <el-row class="myEl-Row">
        <font class="el-rowText">上传成果</font>
      </el-row>
      <div style="height:150px; overflow:auto">
        <el-upload
          class="upload-demo"
          :action=url
          :on-preview="handlePreview"
          :on-remove="handleRemove"
          :before-upload="beforeUpload"
          :before-remove="beforeRemove"
          multiple
          :limit="5"
          :on-exceed="handleExceed"
          :file-list="fileList">
            <el-button size="small" type="primary" v-if="userRole=='incharge'">选择文件</el-button>
            <div slot="tip" class="el-upload__tip" v-if="userRole=='incharge'">只能上传rar/zip文件，且不超过500mb</div>
        </el-upload>
      </div>

      <el-row class="myEl-Row">
        <font class="el-rowText">审核进程</font>
      </el-row>

      <el-table :data="tableData" :show-header='false' max-height="200" style="width: 100%">
        <el-table-column label="日期" width="180">
          <template slot-scope="scope">
            <span style="margin-left: 10px;color:#a5a5a5">{{ scope.row.time }}</span>
          </template>
        </el-table-column>
        <el-table-column label="状态"  width="180">
          <template slot-scope="scope">
            <el-popover v-if="scope.row.status==='未通过'" trigger="hover" placement="top">
              <p>原因: {{ scope.row.reason }}</p>
              <div slot="reference" class="name-wrapper">
                <el-tag size="medium" style="color:red">{{ scope.row.status }}</el-tag>
              </div>
            </el-popover>
            <el-tag v-if="scope.row.status==='已提交'" size="medium">{{ scope.row.status }}</el-tag>
            <el-tag v-if="scope.row.status==='已通过'" style="color:green" size="medium">{{ scope.row.status }}</el-tag>
          </template>
        </el-table-column>
      </el-table>

    </el-main>
</template>

<script>
  export default {
    mounted(){
      this.init()
    },
    data () {
      return {
        info:'',
        endtime:'',
        fileList: [],
        tableData: [],
        url: HOST+'/upload/results'
      }
    },
    methods: {
      handleRemove (file, fileList) {
      },
      handlePreview (file) {
      },
      handleExceed (files, fileList) {
        this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
      },
      beforeRemove (file, fileList) {
        return this.$confirm(`确定移除 ${file.name}？`)
      },
      beforeUpload: function (file) {
        console.log(file)
        // 这里是重点，将文件转化为formdata数据上传
        let fd = new FormData()
        fd.append('file', file)
        console.log(file)
        this.$http.post(HOST+'/upload/results', fd).then(response=> {
          var tag={
            name: file.name.split('.')[0],
            address: 'result/'+file.name,
            commit: new Date()
          }
          this.$http.post(HOST+'/milestones/'+localStorage['milestoneId']+'/results', tag).then(response=> {
            this.$message({
                message: '上传成功',
                type: 'success'
              });
              this.init()
          }).catch(error=> {
            console.log(error.toString())
          })
          console.log(response)
        }).catch(error=> {
          console.log(error.toString())
        })
      },
      init(){
        this.$http.get(
          HOST + '/milestones/' + localStorage["milestoneId"],
          {headers: {'Content-Type': 'application/json;charset=utf-8'}}
          ).then(response=>{
            //console.log(response.data);
            this.info = response.data.info;
            this.endtime = response.data.endTime.slice(0,10);
          }).catch(error=>{
            console.log(error);
          });
          this.tableData=[]
        this.$http.get(
          HOST + '/milestones/' + localStorage["milestoneId"] + '/milestoneHistories',
          {headers: {'Content-Type': 'application/json;charset=utf-8'}}
          ).then(response=>{
            //console.log(response.data);
            for(var i=response.data.length-1;i>=0;i--)
            {
              var sta;
              if (response.data[i].status == -1)
                sta = '未通过'
              else if (response.data[i].status == 0)
                sta = '已提交'
              else if (response.data[i].status == 1)
                sta = '已通过'
              this.tableData.push({
                time: response.data[i].createTime.slice(0,10),
                status: sta,
                reason: response.data[i].reason
              })
            }
          }).catch(error=>{
            console.log(error);
          });
      },
    },
    props:['userRole']
  }
</script>

<style>
  .CheckTime {
    //height:30px;
    width:90px;
    display:block;
    float:left;
    color:#a5a5a5;
  }
</style>
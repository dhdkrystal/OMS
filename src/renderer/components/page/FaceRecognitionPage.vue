<template>
	<div class="facecontent">
		<div class="faceRec-DS" @click="handleClick">
			<img  class="logo-DS" src="@/assets/logo.png">
			<div class="video">
				<video id="video" autoplay="" width="400" height="400"></video>
			</div>
			<canvas id="canvas" class="facePic-DS" width="400" height="400"></canvas>
			<!-- <img  class="facePic-DS" src="@/assets/face-recognition2.png">  -->
		</div>
	</div>
</template>
<script>
	export default {
		name: 'facerecognition-page',
		mounted(){
			this.init()
		},
		methods: {
			init(){
				var video=document.getElementById("video") 
				//var context=canvas.getContext("2d")
				let vm = this
				let context = canvas.getContext('2d')
      		// 浏览器兼容
      		let mediaDevices = navigator.mediaDevices.getUserMedia({ audio: false, video })
      		mediaDevices
      		.then(mediaStream => {
      			video.src = window.URL.createObjectURL(mediaStream)
      			video.onloadedmetadata = (e) => {
      				video.play()
      			}
      			vm.canvas=canvas
      			vm.video = video
      			vm.track = mediaStream.getTracks()[0]
      		})
      		.catch(err => {
      			console.log('err.message' + err.name)
      		})
      		setTimeout(function(){
      			vm.timer=setInterval(function(){ 
      				vm.photo();
      			},1000)
			},3000)
      	},
      	photo(){
      		let vm=this;
      		//console.log(vm.timer)
      		let context=vm.canvas.getContext('2d')
      		context.drawImage(vm.video,0,0,400,400)
      		//导出base64格式的图片数据  
      		var imgData = vm.canvas.toDataURL("image/jpg");
      		//封装blob对象  
      		var blob = vm.dataURItoBlob(imgData);  
			    //组装formdata  
			    var fd = new FormData(); 
			    fd.append('img', blob)
			    console.log(123)
			    console.log(this.$route)
			    this.$http.post(HOST+'/faceRecognition/'+vm.$route.params.userId, fd).then(response=>{
			    	if(response.data.success ===true){
			    		console.log(response)
			    		clearInterval(vm.timer)
			    		vm.track.stop()
			    		var userId=vm.$route.params.userId;
			    		this.$message({
			    			message: '恭喜你，人脸认证成功',
			    			type: 'success'
			    		});
			    		setTimeout(function(){
			    			if(response.data.userRole=='RECEIVER')
			    				vm.$router.replace({ name:'outsourcee',params:{userId}})
			    			else{
			    				vm.$router.replace({ name:'contractee',params:{userId}})
			    			}
			    		},3000);
			    		
			    	}
			    }).catch(error=>{
			    	console.log(error.toString())
			    })
			},
			dataURItoBlob (base64Data) {  
				var byteString;  
				if (base64Data.split(',')[0].indexOf('base64') >= 0)  
					byteString = atob(base64Data.split(',')[1]);  
				else  
					byteString = unescape(base64Data.split(',')[1]);  
				var mimeString = base64Data.split(',')[0].split(':')[1].split(';')[0];  
				var ia = new Uint8Array(byteString.length);  
				for (var i = 0; i < byteString.length; i++) {  
					ia[i] = byteString.charCodeAt(i);  
				}  
				return new Blob([ia], {type: mimeString});  
			},
			handleClick () {
	      //this.$router.replace('/outsourcee/homepage/task')
	  }
	}
}
</script>
<style>
.facecontent{
	width: 100%;
	height:100%;
	background: url(../../assets/face-recognition.png)no-repeat;
	background-size:100% 100%; 	
}
.video{
	background: url(../../assets/face-recognition2.png);
	width: 420px;
	height: 420px;
	background-size: 420px 420px;
}
video{
	margin-top: 10px;
}
.faceRec-DS{
	padding-left: 120px;
	padding-top: 100px;
	width:400px;
	height: 400px;
	text-align: center;
}
.facePic-DS{
	display: none;
	position: fixed;
	right:0;
	top:40px;
}
.logo-DS{
	width: 80px;
	margin: 0 0 20px 0;
}
</style>

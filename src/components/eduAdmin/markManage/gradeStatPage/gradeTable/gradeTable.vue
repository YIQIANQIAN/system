<template>
<div>	
	<div class="positionBar">
		<span>您的当前位置：</span>
		<span><a href="#/login/main/eduAdminHome" class="returnHome">首页</a></span>
		<!-- <span> > <a href="#/login/main/eduAdminHome?eduAdmin" class="returnHome">成绩管理</a></span> -->
		<span> > <a href="#/login/main/eduAdminHome?gradeManage" class="returnHome">成绩</a></span>
		<span> > 成绩统计</span>
	</div>
	<div class="tableSelect">
		<!-- 填选信息进行查询学生成绩 -->
		<select v-model="selGradeType">
			<option disabled value="">选择年制</option>
			<option v-for="gradeTypeOne in gradeType" :value="gradeTypeOne.value">{{gradeTypeOne.text}}</option>
		</select>
		<select v-model="selYearTerm">
			<option disabled value="">选择学期</option>
			<option v-for="yearTermOne in yearTerm" :value="yearTermOne.startYearSemester">{{yearTermOne.startYearSemester}}</option>
		</select>
		<select v-model="selSpeciality">
			<option disabled value="">选择专业</option>
			<option v-for="specialityOne in specialityInfo" :value="specialityOne.specialityId">{{specialityOne.specialityName}}</option>
		</select>
		<select v-model="selCourseName">
			<option disabled value="">选择课程</option>
			<option v-for="courseNameOne in courseInfo" :value="courseNameOne.courseId">{{courseNameOne.courseName}}</option>
		</select>
        <span class="inputFraction">
	        <input v-model="minScore" class="inputGrade">分 — 	<!-- 默认0 -->
	       	<input v-model="maxScore" class="inputGrade">分数	<!-- 默认100 -->
	    </span>
        <button class="am-btn am-btn-success am-radius" @click="inquireBtn()">查询</button>
		<button class="am-btn am-btn-success am-radius" @click="exportBtn()">导出</button>
		<!-- <td class="textBtn"><form action="./exportScoreList" method="get"><button :value="data.courseAssociationId" name="courseAssociationId" type="submit" style="display:visibility;"><a>导出</a></button></form></td> -->
	</div>

	<!-- 成绩单列表 -->
	<div id="gradeTable">
		<div class="gradeTableBody">
			<table class="normalTable">
				<thead>
					<tr>
						<th>学号</th>
						<th>姓名</th>
						<th>年级</th>
						<th>专业</th>
						<th>学期</th>
						<th>课程</th>
						<th>分数</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="data in scoreList">
						<td v-text="data.studentId"></td>
						<td v-text="data.studentName"></td>
						<td v-text="data.term"></td>
						<td v-text="data.specialityName"></td>
						<td v-text="data.stuSemester"></td>
						<td v-text="data.courseName"></td>
						<td v-text="data.grade"></td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>

	<!-- 错误弹窗提示 -->
	<Modal v-model="modalResult" id="modalBody" :styles="{top:'10rem'}">
		<div style="text-align:center; font-size:1.1rem;">
		    <p v-if="resultBool === '1'">未找到所查询内容！</p>
		    <p v-else-if="resultBool === '2'">未找到可下载的内容！</p>
		    <p v-else-if="resultBool === '3'">请选择年制！</p>
		    <p v-else-if="resultBool === '4'">请选择后进行查询！</p>
		</div>
	    <div slot="footer" style="text-align:center;">
	        <Button id="modalBtn" @click="resultOk()">确认</Button>
	    </div>
	</Modal>
</div>
</template>

<script>
export default {
	name: 'gradeTable',
	data () {
		return {
			// 全局变量定义
			selGradeType: '',
			selYearTerm: '',
			selSpeciality: '',
			selCourseName: '',
			gradeType: [
				{text: '三年制', value: '3'},
				{text: '五年制', value: '5'}
			],
			yearTerm: [
				// "2016-2017学年第一学期"
			],
			specialityInfo: [],
			courseInfo: [
				// {courseName: '护理学', courseId: '123456'}
			],
			minScore: '',
			maxScore: '',
			// 返回学生成绩列表
			scoreList: [
				// {stuNum: '20142201010', stuName: '何平', stuGrade: '大二', stuMajor: '护理学', stuSemester: '2016-2017第一学期', stuCourse: '护理学', stuScore: '80'}
				// {},{},{}
			],
			modalResult: false,
			resultBool: ''
		}
	},
	// 页面初始化，获取学期、专业、课程下拉数据
	beforeMount: function() {
		this.$http.post('./getYearTermList',{},{
            "Content-Type":"application/json"
        }).then(function(response){
            console.log("获取申请:");
            console.log(response.body);
            var data = response.body;
            this.yearTerm = data.yearTerm;
        },function(error){
            console.log("获取申请error:");
            console.log(error);
        });
        this.$http.post('./specialityList',{},{
            "Content-Type":"application/json"
        }).then(function(response){
            console.log("获取申请:");
            console.log(response.body);
            var data = response.body;
            this.specialityInfo = data.specialityInfo;
        },function(error){
            console.log("获取申请error:");
            console.log(error);
        });
        this.$http.post('./courseManage/getCourseAndClassInfo',{},{
            "Content-Type":"application/json"
        }).then(function(response){
            console.log("获取申请:");
            console.log(response.body);
            var data = response.body;
            this.courseInfo = data.courseInfo;
        },function(error){
            console.log("获取申请error:");
            console.log(error);
        });
    },
	methods: {
    	// 查询按钮
		inquireBtn: function() {
			if (this.minScore == '') {
    			this.minScore = '0';
    		}
    		if (this.maxScore == '') {
    			this.maxScore = '100';
    		}
    		if (this.selGradeType == "") {
    			this.modalResult = true;
    			this.resultBool = '3';
    		}else if (this.selYearTerm == "" || this.selCourseName == "" || this.selSpeciality == "") {
    			this.modalResult = true;
    			this.resultBool = '4';
    		}else {
	    		this.$http.post('./findScore',{
		        	"gradeType": this.selGradeType,
		        	"yearTerm": this.selYearTerm,
		        	"specialityId": this.selSpeciality,
		        	"courseId": this.selCourseName,
		        	"minScore": this.minScore,
		        	"maxScore": this.maxScore
		        },{    
		            "Content-Type":"application/json"
		        }).then(function(response){
		            console.log("获取申请:");
		            console.log(response.body);
		            var data = response.body;
		            if (data.scoreList != []) {
		            	this.scoreList = data.scoreList;
		            }else{
				        this.modalResult = true;
				        this.resultBool = '1';
				    }
		        },function(error){
		            console.log("获取申请error:");
		            console.log(error);
	        	});
    		}
    		
    	},
    	// 导出按钮
		exportBtn: function () {
    		if (this.minScore == '') {
    			this.minScore = '0';
    		}
    		if (this.maxScore == '') {
    			this.maxScore = '100';
    		}
    		if (this.selGradeType == "") {
    			this.modalResult = true;
    			this.resultBool = '3';
    		}else if (this.selYearTerm == "" || this.selCourseName == "" || this.selSpeciality == "") {
    			this.modalResult = true;
    			this.resultBool = '4';
    		}else {
				location.href = "./exportScoreListByMaxMinScore?gradeType="+this.selGradeType+"&"+"yearTerm="+this.selYearTerm+"&"+"specialityId="+this.selSpeciality+"&"+"courseId="+this.selCourseName+"&"+"minScore="+this.minScore+"&"+"maxScore="+this.maxScore;
			}
  		},
    	// 弹窗提示点击确定，弹窗消失
    	resultOk: function () {
    		this.modalResult = false;
    	}
	}
}
</script>

<style scoped>
.inputGrade {
	width: 2.3rem;
}
.inputFraction {
	margin-right: 1.4rem;
	font-size: 0.8rem;
}

#gradeTable {
	background-color: #f3f3f3;
	width: 100%;
	text-align: center;
}
.gradeTableBody {
	padding: 1rem 5rem;
}
.gradeTableBody select {
	margin: 0 1rem;
}
/*.gradeTableBody table {
	margin-bottom: 1.6rem;
}*/

</style>

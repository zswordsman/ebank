<template>
	<div class="welcome">
		<div class="mdui-typo">
			<blockquote class="blockquote_normal">
				<p>注：仅统计[用户身份]的流水收入、支出，包括所有钱包</p>
			</blockquote>
		</div>
		<div id="statistics" style="width:100%;height:500px"></div>
	</div>
</template>
<script>
	// let echarts = require('echarts');
	let echarts = require('echarts/lib/echarts');
	// 引入柱状图
	require('echarts/lib/chart/bar');
	require('echarts/lib/chart/line');
	// 引入提示框和标题组件
	require('echarts/lib/component/tooltip');
	require('echarts/lib/component/title');
	require("echarts/lib/component/legendScroll");
	require("echarts/lib/component/dataZoom");
	require("echarts/lib/component/markPoint");
	require("echarts/lib/component/markLine");
	require("echarts/lib/component/toolbox");
	
	export default {
		data(){
			return {
				statistics : '',
			};
		},
		mounted(){
			let t = this;
			
			t.statistics = echarts.init(document.getElementById('statistics'));
			let order_options = {
				title : {
					left: 'center',
					text: '近期 15 天每日用户身份收入、支出金额统计',
					subtext: '按日划分，金额单位：分'
				},
				tooltip : {
					trigger: 'axis'
				},
				legend: {
					top : '60px',
					data: []
				},
				// color : ['#7079DF','#D15B7F','#FB6E6C','#FF9F69','#FEB64D','#FFDA43','#FFE88E'],
				toolbox: {
					show : true,
					feature : {
						magicType : {show: true, type: ['line', 'bar']},
						saveAsImage : {show: true}
					}
				},
				grid : {
					top : '150px',
					bottom : '50px',
					containLabel: true
				},
				calculable : true,
				dataZoom: [
					{
						show: true,
						realtime: true,
						start: 80,
						end: 100
					},
					{
						type: 'inside',
						realtime: true,
						start: 60,
						end: 100
					}
				],
				xAxis : [
					{
						type : 'category',
						nameGap : '60',
						data : []
					}
				],
				yAxis : [
					{
						type : 'value'
					}
				],
				series : []
			};
			get('/user_transfer',{},function(data){
				let dates = data.dates;
				let purse_types = data.purse_types;
				let out = data.out;
				let into = data.into;
				let amounts = data.amounts;
				
				// X 轴数据展示
				order_options.xAxis[0].data = dates;
				
				// 基本信息变量
				let series_template = function(){
					return {
						name:'',
						type:'bar',
						stack:'',	// 正负轴相同的值会上下顶在一起
						data:[],
						// markPoint : {
						// 	symbol : 'pin',
						// 	data : [
						// 		{type : 'max', name: '最大值'},
						// 		{type : 'min', name: '最小值'}
						// 	]
						// },
						markLine : {
							data : [
								{type : 'average', name: '平均值'}
							]
						},
						label: {
							show: true,
						}
					};
				};
				
				
				for(let type in amounts){
					let series = series_template();
					series.name = purse_types[type];
					order_options.legend.data.push(series.name);
					series.stack = type.substr(0,1) == '-' ? type.substr(1) : type;
					for(let date in amounts[type]){
						let date_val = amounts[type][date];
						series.data.push({
							value : date_val,
							// 在柱状条上显示数值
							label : {
								normal: {
									show: true,
									rotate: 90,
								}
							},
						});
						// if(date_val < 0){
						// 	series.markPoint.symbolRotate = 180;
						// }
					}
					order_options.series.push(series);
				}
				
				t.statistics.setOption(order_options);
			});
			
			window.onresize = function(){
				setTimeout(function(){
					t.statistics.resize();
				},200);
			};
		}
	}
</script>
<template>
	<div>
		<!-- Form -->
		<el-button id="addBtn" type="info" @click="dialogFormVisible = true" icon="el-icon-plus">添加日程</el-button>
		<el-dialog title="新增排班" :visible.sync="dialogFormVisible" width="30%">
			<el-form :model="form" label-width="100px">
				<el-form-item label="员工姓名">
					<el-input></el-input>
				</el-form-item>
				<el-form-item label="员工职位">
					<el-input></el-input>
				</el-form-item>
				<el-form-item label="工作开始时间">
					<el-input></el-input>
				</el-form-item>
				<el-form-item label="工作结束时间">
					<el-input></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="dialogFormVisible = false">取 消</el-button>
				<el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
			</div>
		</el-dialog>
		<FullCalendar class="fullCalendar" ref="fullCalendar" :options="calendarOptions" />
	</div>
</template>

<script>
import FullCalendar from '@fullcalendar/vue'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import timeGridPlugin from '@fullcalendar/timegrid'
// import { Calendar } from '@fullcalendar/core'
import resourceTimeGridPlugin from '@fullcalendar/resource-timegrid'

import axios from 'axios'

export default {
	components: {
		FullCalendar // 像使用自定义组件一样使用
	},
	data() {
		return {
			calendarApi: null,
			calendarOptions: {
				plugins: [dayGridPlugin, interactionPlugin, timeGridPlugin, resourceTimeGridPlugin],
				// plugins: [dayGridPlugin, interactionPlugin, timeGridPlugin, resourceTimeGridPlugin], // 需要用哪个插件引入后放到这个数组里
				initialDate: new Date(), // 日历第一次加载时显示的初始日期。可以解析为Date的任何职包括ISO8601日期字符串，例如"2014-02-01"。
				initialView: 'dayGridMonth', // 日历加载时的初始视图，默认值为'dayGridMonth'，可以为任何可用视图的值，如例如'dayGridWeek'，'timeGridDay'，'listWeek'
				headerToolbar: {
					// 在日历顶部定义按钮和标题。将headerToolbar选项设置为false不会显示任何标题工具栏。可以为对象提供属性start/center/end或left/center/right。这些属性包含带有逗号/空格分隔值的字符串。用逗号分隔的值将相邻显示。用空格分隔的值之间会显示一个很小的间隙。
					right: 'today prev,next',
					center: 'title',
					// left: 'dayGridMonth,dayGridWeek,dayGridDay'
					left: 'dayGridMonth,timeGridWeek,timeGridDay'
				},
				/*  日程选项start  */
				allDaySlot: false,
				// axisFormat: 'h(:mm)tt',
				slotLabelFormat: {
					hour: '2-digit',
					minute: '2-digit',
					meridiem: false,
					hour12: false // 设置时间为24小时
				},
				slotMinutes: '00:30:00',
				scrollTime: '18:00:00',
				nowIndicator: false, // 是否显示当前时间的指示条
				slotMinTime: '06:00:00', // 图表左侧展示的开始时间
				slotMaxTime: '24:00:00', // 图表左侧展示的结束时间
				slotDuration: '00:30:00', // 左侧时间标尺的间隔
				/*  日程选项end */
				/* 日程事件数据start */
				editable: true,
				slotEventOverlap: true, // 事件显示是否可覆盖
				/* 日程事件数据end */
				locale: 'zh-cn', // 设置日历的语言，中文为 “zh-cn”
				firstDay: '1', // 设置每周的第一天，默认值取决于当前语言环境，该值为代表星期几的数字，且值必须为整数，星期日=0
				weekNumberCalculation: 'ISO', // 指定"ISO"结果为ISO8601周数。指定"ISO"将firstDay的默认值更改为1（Monday）
				buttonText: {
					// 文本将显示在headerToolbar / footerToolbar的按钮上。不支持HTML注入。所有特殊字符将被转义。
					today: '今天',
					month: '月',
					week: '周',
					day: '天'
				},
				eventTimeFormat: {
					// 在每个事件上显示的时间的格式
					hour: 'numeric',
					minute: '2-digit',
					hour12: false
				},
				// customButtons: {
				// 	// 定义可在headerToolbar / footerToolbar中使用的自定义按钮
				// 	today: {
				// 		text: '今天', // 按钮的展示文本
				// 		click: this.todayClick // 点击按钮触发的事件，这里要注意的是当按钮绑定了事件之后该按钮原本自带的事件将会失效
				// 	},
				// 	next: {
				// 		click: this.nextClick
				// 	},
				// 	prev: {
				// 		click: this.prevClick
				// 	}
				// },
				events: null, // 事件（排班）列表
				eventClick: this.handleDateClick, // 点击事件时，触发该回调
				eventMouseEnter: this.handleMouseEnter, // 鼠标悬停在事件上时，触发该回调
				eventMouseLeave: this.handleMouseLeave, // 鼠标移除时，触发该回调
				eventDrop: this.handleEventDrop, // 拖动日程，触发该回调
				dateClick: this.handleDateClick // 当用户单击日期或时间时,触发该回调，触发此回调，您必须加载interaction插件
			},
			// 排班信息相关属性
			schedulingInfo: [
				// 将在日历上显示的事件对象， events 可以是数组、json、函数。具体可以查看官方文档
				{
					// 员工id、门店id、班次起止时间、
					title: 'event1',
					id: 1,
					start: '2023-02-14 08:00:00',
					end: '2023-02-14 14:00:00', // 这里要注意，end为可选参数，无end参数时该事件仅在当天展示
					backgroundColor: '#FDEBC9', // 该事件的背景颜色
					borderColor: '#FDEBC9', // 该事件的边框颜色
					textColor: '#F9AE26' // 该事件的文字颜色
				},
				// 添加事件的表单
				{
					// 员工id、门店id、班次起止时间、
					title: 'event2',
					id: 2,
					start: '2023-02-14 09:00:00',
					end: '2023-02-14 15:00:00' // 这里要注意，end为可选参数，无end参数时该事件仅在当天展示
				},
				{
					// 员工id、门店id、班次起止时间、
					title: 'event3',
					id: 3,
					start: '2023-02-14 10:00:00',
					end: '2023-02-14 16:00:00', // 这里要注意，end为可选参数，无end参数时该事件仅在当天展示
					backgroundColor: '#FDEBC9', // 该事件的背景颜色
					borderColor: '#FDEBC9', // 该事件的边框颜色
					textColor: '#F9AE26' // 该事件的文字颜色
				},
				{
					// 员工id、门店id、班次起止时间、
					title: 'event4',
					id: 4,
					start: '2023-02-15 09:00:00',
					end: '2023-02-15 15:00:00' // 这里要注意，end为可选参数，无end参数时该事件仅在当天展示
				},
				{
					title: 'test',
					id: 5,
					employeName: '张三',
					position: 0,
					start: '2023-02-16 10:00:00',
					end: '2023-02-16 14:00:00'
				}
			],
			// 添加事件的对话框表单
			dialogFormVisible: false,
			form: {}
		}
	},
	mounted() {
		// 这里有两点要注意，想要调用插件的方法，要在组件上设置ref
		// 并且在组件未加载的时候this.$refs中是没有fullCalendar的，所以未加载的时候调用方法会报错
		this.calendarApi = this.$refs.fullCalendar.getApi()
		this.calendarOptions.events = this.schedulingInfo
		this.initSchedulingInfo()
	},
	methods: {
		// 从后台获取当前排表信息
		async initSchedulingInfo() {
			console.log('获取排班信息')
			let result = await axios({
				url: 'xxx',
				methods: 'get'
			})
			this.schedulingInfo = result
		},
		handleDateClick(dateClickInfo) {
			console.log(dateClickInfo)
		},
		handleMouseEnter(mouseEnterInfo) {
			console.log(mouseEnterInfo.event.startStr)
			// 提示:mouseEnterInfo.event.startStr 可以获取当前事件的开始事件
		},
		handleMouseLeave(mouseEnterInfo) {
			console.log(mouseEnterInfo)
		},
		handleEventDrop(eventDropInfo) {
			console.log('触发了拖动日程回调')
			console.log(eventDropInfo)
		}
	}
}
</script>
<style scoped>
#addBtn {
	position: absolute;
	top: 8px;
	left: 130px;
}
</style>

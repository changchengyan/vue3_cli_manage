<template>
	<div class="block">
		<el-form :inline="true" :model="ruleForm" :rules="rules" ref="ruleForm" label-width="165px" class="ruleForm">
			<el-form-item v-for="(item, key) in formList" :key="key" :label="item.label" :prop="item.name">
				<!-- 下拉框 -->
				<el-select v-if="item.type === 1" v-model="ruleForm[item.name]" :placeholder="item.placeholder" :disabled="item.disabled">
					<el-option v-for="(it, k) in item.children" :key="k" :label="it.label" :value="it.value"></el-option>
				</el-select>
				<!-- 单选框 -->
				<template v-else-if="item.type === 2">
					<el-radio v-for="(it, k) in item.children" :key="k" v-model="ruleForm[item.name]" :label="it.value">{{ it.label }}</el-radio>
				</template>
				<!-- 日期选择 -->
				<template v-else-if="item.type === 3">
					<el-date-picker v-model="ruleForm[item.name]" type="month" :placeholder="item.placeholder" value-format="yyyyMM"></el-date-picker>
				</template>
				<!-- 文本框 -->
				<el-input v-else v-model="ruleForm[item.name]" :placeholder="item.placeholder" :disabled="item.disabled"></el-input>
			</el-form-item>
			<!-- 上传预览图片 -->
			<dse-upload
				:fileList="fileList"
				:files="files"
				:isUploadDisabled="isUploadDisabled"
				@onHandlePictureSuccess="onHandlePictureSuccess"
				@onHandlePictureCardRemove="onHandlePictureCardRemove"
			/>
			<!-- form 提交按钮 -->
			<div class="actions">
				<span class="save" @click="onHandleSubmitForm('ruleForm')">保存</span>
				<span class="cancel" @click="onHandleCancel">取消</span>
			</div>
			<!-- <el-form-item class="formBtns">
				<el-button @click="onHandleCancel">返&nbsp;回</el-button>
				<el-button type="primary" @click="onHandleSubmitForm('ruleForm')">保&nbsp;存</el-button>
			</el-form-item> -->
		</el-form>
	</div>
</template>

<script>
// 新增蓄水池
import { mapGetters } from 'vuex';
import DseUpload from '../../../../common/components/DseUpload';
import { systemAction } from '../../../../mixins/system';
import { VDT } from '../../../../utils/el_validater';

export default {
	props: {
		data: {
			type: Object,
			default() {
				return {};
			}
		},
		onHandleCancel: {
			type: Function,
			default() {
				return () => {};
			}
		},
		onHandleSubmit: {
			typp: Function,
			default() {
				return () => {};
			}
		}
	},
	mixins: [systemAction],
	computed: mapGetters({
		get_partition: 'root_store/get_partition',
		fileList: 'system_store/GET_ENGINEERING_FILELIST'
	}),
	components: {
		DseUpload
	},
	data() {
		return {
			name: 'systemEngineeringIndex',
			formList: [],
			ruleForm: {},
			rules: {},
			// 图片 data
			files: [],
			isUploadDisabled: true
		};
	},
	methods: {
		// 上传图片成功后返回
		onHandlePictureSuccess(file) {
			const that = this;

			const { data = {} } = that;
			const { prcd } = data;
			console.warn(prcd);

			file.fileBiz = prcd;
			that.files.push(file);
		},
		// 上传图片成功后删除
		onHandlePictureCardRemove(file = {}) {
			const that = this;

			const { data } = that;
			let { isEdit } = data;
			
			if (file.fileBiz && file.fileId) isEdit = true;
			// 删除图片
			const { fileId = '' } = file;
			that.$store.dispatch('system_store/handleEngineeringRemoveFile', {
				fileId,
				isEdit,
				that,
				callback(status) {
					if (status) {
						that.$message.success('图片删除成功~');
					} else {
						that.$message.warning('图片删除失败!');
					}
				}
			});
		},
		onHandleSubmitForm(formName) {
			const that = this;

			const { data = {} } = that;
			const { prcd = '' } = data;
			const { ruleForm, files = [] } = that;

			that.$refs[formName].validate(valid => {
				if (valid) {
					that.$emit('onHandleSubmit', { ...ruleForm, files: files.map((item = {}) => ({ ...item, fileBiz: prcd })) });
				} else {
					console.log('error submit!!');
					return false;
				}
			});
		},
		onHandleResetForm(formName) {
			const that = this;

			that.$refs[formName].resetFields();
			that.onHandleBack();
		},
		onHandleBack() {
			const that = this;

			that.$router.push({
				name: 'systemEngineeringIndex'
			});
			that.$store.commit('root_store/set_asideStatus', true);
			that.$emit('root_store/showAside');
		},
		// 参数回填
		_backfilling() {
			const that = this;

			const { data } = that;
			const { isEdit } = data;
			if (isEdit) {
				Object.keys(data).forEach((key = '') => {
					if (!data[key]) {
						Object.assign(data, {
							[key]: ''
						});
					}
				});
				/*
					预计出参：详情数据
					loc 			所在地
					prtypenm 		工程类型名称
					prtype 			工程类型
					gldw 			管理单位
					adcd 			所属分区
					lttd 			经度
					adnm 			所属分区名称
					cwsnm 			所属供水工程名称
					prnm 			工程名称
					prcd 			工程编码
					runCondnm 		运行状况：1在用良好，0停用
					lgtd 			纬度
					compym 			建站年月
					deadV 			死库容
					deadZ 			死水位
					totV 			总库容
					frscV 			调洪库容
					dszfl 			设计规模（万方/天）
					desHead 		设计扬程
					dsincp 			装机容量(kW)
				*/
				/*
					实际出参：详情数据
					{
						"loc": "",
						"prtypenm": "",
						"deadV": "",
						"deadZ": "",
						"dszfl": "",
						"prtype": "",
						"gldw": "",
						"frscV": "",
						"adcd": "",
						"lttd": "",
						"totV": "",
						"dsincp": "",
						"desHead": "",
						"adnm": "",
						"cwsnm": "",
						"prnm": "",
						"prcd": "",
						"runCondnm": "",
						"lgtd": "",
						"compym": "",
						"isEdit": ""
					}
				*/
				const {
					loc,
					// prtypenm,
					volumn,
					sckzsw,
					// prtype,
					gldw,
					adcd,
					lttd,
					scbg,
					cwsnm,
					height,
					width,
					prnm,
					prcd,
					runCond,
					lgtd,
					compym,

					files
				} = data;

				/*
				入参：详情数据
				poolcd 			//	蓄水池代码
				poolnm 			//	蓄水池名称
				cwsCd 			//	所属供水工程
				loc 			//	所在地
				compYm 			//	建成年月格式：yyMMdd
				engMan 			//	管理单位
				lgtd 			//	经度单位
				lttd 			//	纬度单位
				height 			//	蓄水池高度
				width 			//	蓄水池宽度
				volumn 			//	蓄水池容积
				scbg 			//	蓄水池标高
				sckzsw 			//	蓄控制水位
				runCond 		//	运行状况：1在用良好，0停用
				ts 	 			//	时间戳
				nt 				//	备注
				adcd 			//	所属分区
				*/
				that.ruleForm = {
					...that.ruleForm,
					poolcd: prcd + '',
					poolnm: prnm + '',
					cwsCd: cwsnm + '',
					loc: loc + '',
					compYm: compym,
					engMan: gldw + '',
					lgtd: lgtd + '',
					lttd: lttd + '',
					height: height,
					width: width,
					volumn: volumn,
					scbg: scbg + '',
					sckzsw: sckzsw + '',
					runCond: runCond + '',
					ts: '' + '',
					nt: '' + '',
					adcd: adcd + ''
				};

				that.formList.find((item = {}) => {
					if (item.name === 'poolcd') {
						item.disabled = true;
						return true;
					} else {
						item.disabled = false;
					}
				});

				// 填充图片
				that.$store.dispatch('system_store/handleEngineeringFileList', {
					fileBiz: prcd,
					that
				});

				// 初始化编辑开启上传
				that.isUploadDisabled = false;
			} else {
				// 初始化图片为空
				that.$store.commit('system_store/SET_ENGINEERING_FILELIST', []);
				
				// 初始化新增禁用上传
				that.isUploadDisabled = true;
			}
		},
		// 初始化分区
		_initPartition() {
			const that = this;

			const { cwsCd } = that.data;

			that.formList.find((item = {}) => {
				if (item.name === 'cwsCd') {
					const children = that.get_partition;
					const [{ value }] = children;
					cwsCd ? (that.ruleForm[item.name] = cwsCd) : (that.ruleForm[item.name] = value);
					item.children = children;
					return true;
				}
			});
		},
		_initialization() {
			const that = this;

			const formList = [
				{
					name: 'poolcd',
					label: '站点代码'
				},
				{
					name: 'cwsCd1',
					label: '站点类型',
					disabled: true,
					type: 1,
					value: '0',
					children: [
						{
							label: '蓄水池',
							value: '0'
						}
					]
				},
				{
					name: 'poolnm',
					label: '站点名称'
				},
				{
					name: 'runCond',
					label: '是否启用',
					type: 2,
					value: '1',
					children: [
						{
							label: '启用',
							value: '1'
						},
						{
							label: '停用',
							value: '0'
						}
					]
				},
				{
					name: 'lgtd',
					label: '经度'
				},

				{
					name: 'compYm',
					label: '建站年月',
					type: 3,
					value: ''
				},
				{
					name: 'lttd',
					label: '纬度'
				},
				{
					name: 'engMan',
					label: '管理单位'
				},
				{
					name: 'volumn',
					label: '蓄水池容积(m³)'
				},
				{
					name: 'sckzsw',
					label: '蓄水池控制水位(m)'
				},
				{
					name: 'cwsCd',
					label: '所属工程',
					type: 1,
					value: '0',
					children: []
				},
				{
					name: 'loc',
					label: '地址'
				}
			].map((item = {}) => ({
				type: 0,
				children: [],
				placeholder: `${item.type ? '请选择' : '请输入'}${item.label}`,
				disabled: false,
				...item,
				label: item.label + ' : '
			}));

			// 初始化表单验证
			const COMMON_RULES_CONFIG = {
				required: false,
				trigger: 'blur'
			};
			const rules = {
				poolcd: [
					{
						...COMMON_RULES_CONFIG,
						required: true,
						message: '请输入1-12字母或者数字!',
						validator(rule, value, callback) {
							if (VDT.code(value)) callback();
							else callback(new Error(this.message));
						}
					}
				],
				poolnm: [{ ...COMMON_RULES_CONFIG, required: true }],
				lgtd: [
					{
						...COMMON_RULES_CONFIG,
						required: true,
						message: '您输入的经度有误或者最多保存六位小数',
						validator(rule, value, callback) {
							if (VDT.lgtd(value)) {
								callback();
							} else {
								callback(new Error(this.message));
							}
						}
					}
				],
				lttd: [
					{
						...COMMON_RULES_CONFIG,
						required: true,
						message: '您输入的纬度有误或者最多保存六位小数',
						validator(rule, value, callback) {
							if (VDT.lttd(value)) {
								callback();
							} else {
								callback(new Error(this.message));
							}
						}
					}
				],
				compYm: [COMMON_RULES_CONFIG],
				engMan: [COMMON_RULES_CONFIG],
				adcd: [{ ...COMMON_RULES_CONFIG, required: true }],
				loc: [COMMON_RULES_CONFIG],
				height: [COMMON_RULES_CONFIG],
				width: [COMMON_RULES_CONFIG],
				sckzsw: [COMMON_RULES_CONFIG]
			};

			const ruleForm = {};
			formList.forEach((item = {}) => {
				const { name = '', value, placeholder = '' } = item;

				Object.assign(ruleForm, {
					[name]: value
				});

				if (Array.isArray(rules[name])) {
					const [first, ...others] = rules[name];

					rules[name] = [
						{
							message: placeholder,
							...first
						},
						others
					];
					// 修改作用域 bug
					// rules[name][0].message = placeholder;
				}
			});

			that.formList = formList;
			that.rules = rules;
			that.ruleForm = ruleForm;

			that.$nextTick(() => {
				// 参数回填
				that._backfilling();
				// 初始化分区
				that._initPartition();
			});
		}
	},
	mounted() {
		const that = this;

		that._initialization();
	}
};
</script>

<style scoped="scoped" lang="scss"></style>

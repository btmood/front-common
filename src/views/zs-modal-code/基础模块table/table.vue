<template>
	<div id="home-main" class="home-main">
		
		<div class="zs-btn-content">
			<div class="zs-search-widh">
				<div class="search-item">
					<Input placeholder="银行名称、开户行、账户、持有人" v-model="searchContent" icon="ios-search" @on-enter="search" @on-click="search" />
					<div class="btn-search" @click="search">搜索</div>
				</div>
			</div>
			<div class="zs-btn-list">
				<div class="btn-compose" @click="add" style="padding-right: 0;">
					<img src="../../../../images/newsimg/tianjia.png" style="width: 16px;" />
					<span>新增</span>
				</div>
				<div class="btn-compose" @click="del" style="padding-left: 0;">
					<img src="../../../../images/newsimg/shanchu.png" style="width: 16px;" />
					<span>删除</span>
				</div>
			</div>
		</div>
		
		
		<Table class="zs-table-content" :loading="loading" size="small" border stripe ref="selection" :columns="columns"
		 :data="dataList" @on-selection-change="selectionChange" @on-row-dblclick="edit"></Table>
		<div class="zs-page-content">
			<div style="float: right;">
				<Page :total="dataCount" :page-size="pageSize" @on-change="changePage" @on-page-size-change="changePageSize"
				 show-total show-elevator show-sizer transfer></Page>
			</div>
		</div>
		<Modal :width="920" v-model="editStoreModal" title="编辑" :mask-closable="false" class="zs-form-content" >
			<Form class="zs-form-content" ref="formItem" :model="formItem" :label-width="100" :rules="ruleValidate" style="margin-top: 16px;">
				<Row>
					<Col span="12">
					<FormItem label="银行名称" prop="bankName">
						<Input v-model="formItem.bankName" placeholder="请填写"></Input>
					</FormItem>
					</Col>
					<Col span="12">
					<FormItem label="开户行" prop="bankOpen">
						<Input v-model="formItem.bankOpen" placeholder="请填写"></Input>
					</FormItem>
					</Col>
				</Row>
				<Row>
					<Col span="12">
					<FormItem label="账户" prop="account">
						<Input v-model="formItem.account" placeholder="请填写"></Input>
					</FormItem>
					</Col>
                    <Col span="12">
					<FormItem label="持有人" prop="cardHolder">
						<Input v-model="formItem.cardHolder" placeholder="请填写"></Input>
					</FormItem>
					</Col>
				</Row>
			</Form>
			<div slot="footer">
				<Button type="primary" @click="save">保存</Button>
			</div>
		</Modal>

	</div>
</template>

<script>
	import Util from '@/libs/util';
	import zsCommon from '@/libs/common';
	export default {
		name: 'ZsImsBaseStoreTable',
		data() {
			return {
				hasDel: false,
				//仓库类型集合
				storeTypeList: [],
				tempCode: '',
				treeData: [],
				editStoreModal: false,
				formItem: {},
				ruleValidate: {
					bankName: [{
						type: 'string',
						required: true,
						trigger: 'blur',
						message: '银行名称为空'
					}],
                    bankOpen: [{
						type: 'string',
						required: true,
						trigger: 'blur',
						message: '开户行为空'
					}],
                    account: [{
						type: 'string',
						required: true,
						trigger: 'blur',
						message: '账户为空'
					}],
					cardHolder: [{
						type: 'string',
						required: true,
						trigger: 'blur',
						message: '持有人为空'
					}],
				},
				columns: [{
						type: 'selection',
						width: 60,
						align: 'center'
					},
                    {
						type: 'index',
						title: '序号',
						width: 70,
						resizable: true,
						tooltip: true,
						align: 'center',
					},
					{
						title: '银行名称',
						key: 'bankName',
						width: 150,
						align: 'left',
					},
					{
						title: '开户行',
						key: 'bankOpen',
						width: 150,
						align: 'left',
					},
					{
						title: '账户',
						key: 'account',
						width: 150,
						align: 'left',
					},
					{
						title: '持有人',
						key: 'cardHolder',
						width: 150,
						align: 'left',
					},

				],
				dataList: [],
				// 初始化信息总条数
				dataCount: 0,
				// 每页显示多少条
				pageSize: 10,
				//当前页码
				pageIndex: 1,
				loading: true,
				selectionArr: [],
				searchContent: undefined
			}
		},
		methods: {
			save() {
				this.$refs['formItem'].validate((valid) => {
					if (valid) {
						if (this.formItem.id == undefined) {
							Util.ajax({
								method: 'POST',
								url: '/zs-base/zsBaseBankAccount/add',
								data: {
									data: this.formItem
								}
							}).then(res => {
								if (res.data.code == zsCommon.CODE_SUCCESS) {
									this.$Message.info(zsCommon.INFO_SAVE_SUCCESS);
									this.getData()
								} else {
									this.$Message.error(zsCommon.INFO_SAVE_ERROR);
								}
								this.editStoreModal = false;
							});
						} else {
							Util.ajax({
								method: 'POST',
								url: '/zs-base/zsBaseBankAccount/update',
								data: {
									data: this.formItem
								}
							}).then(res => {
								if (res.data.code == zsCommon.CODE_SUCCESS) {
									this.$Message.info(zsCommon.INFO_SAVE_SUCCESS);
									this.getData()
								} else {
									this.$Message.error(zsCommon.INFO_SAVE_ERROR);
								}
								this.editStoreModal = false;
							});
						}
					}
				})
			},
			//获取列表数据
			getData(currentPage, pageSize) {
				var perParams = {};
				if (this.searchContent != undefined && this.searchContent != '') {
					perParams.searchContent = this.searchContent;
				}
				this.loading = true;
				Util.ajax({
					method: 'POST',
					url: '/zs-base/zsBaseBankAccount/query',
					data: {
						currentPage: currentPage,
						pageSize: pageSize,
						perParams: perParams
					}
				}).then(res => {
					this.loading = false;
					if (res.data.code == zsCommon.CODE_SUCCESS) {
						this.dataCount = res.data.data.totalNum;
						this.dataList = res.data.data.items;
						
						this.hasDel = res.data.hasDel;
					}
				});
			},
			search() {
				this.pageIndex = 1;
				this.getData(this.pageIndex, this.pageSize);
			},
			//分页控件分页
			changePage(index) {
				this.pageIndex = index;
				this.getData(index, this.pageSize);
			},
			//分页控件切换页面数据条数
			changePageSize(pageSize) {
				this.pageSize = pageSize;
				this.getData(this.pageIndex, this.pageSize);
			},
			//选中/取消列表数据时触发，返回当前列表选中的数据数组
			selectionChange(selection) {
				this.selectionArr = selection;
			},
			// 新增
			add() {
				this.tempCode = '';
				this.formItem = {};
				this.$refs.formItem.resetFields();
				this.editStoreModal = true;
			},
			// 删除
			del() {
				if (this.selectionArr.length == 0) {
					this.$Message.warning(zsCommon.WARNING_SELECTION_NULL);
					return;
				}
				//获得选中数据的数据的ID集合
				var ids = [];
				for (var idObj of this.selectionArr) {
					ids.push(idObj.id);
				}
				this.$Modal.confirm({
					title: zsCommon.WARNING_TITLE,
					content: zsCommon.WARNING_DELETE,
					onOk: () => {
						Util.ajax({
							method: 'POST',
							url: '/zs-base/zsBaseBankAccount/delete',
							data: {
								data: {
									ids: ids
								}
							}
						}).then(res => {
							if (res.data.code == zsCommon.CODE_SUCCESS) {
								this.$Message.info(zsCommon.WARNING_DELETE_SUCCESS);
								this.getData(this.pageIndex, this.pageSize);
							} else {
								this.$Message.error(zsCommon.WARNING_DELETE_ERROR);
							}
						});
					}
				});
			},
			// 修改
			edit(row, index) {
				this.formItem = row;
				this.tempCode = row.code;
				this.editStoreModal = true;
			},
		},
		mounted() {
			let height = document.documentElement.clientHeight || document.body.clientHeight;
			let mengHeight = height -94;
			document.getElementById('home-main').style.height = mengHeight + 'px';
			
			this.getData(1, this.pageSize);
		}
	}
</script>

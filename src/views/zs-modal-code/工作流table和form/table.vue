<template>
    <div id="home-main" class="home-main">
        <div class="zs-btn-content">
			<div class="zs-search-widh">
				<div class="search-item">
					<Input placeholder="模糊搜索" v-model="searchContent" icon="ios-search" @on-enter="search"
						@on-click="search" />
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
        <div style="position: relative;">
			<Table class="zs-table-content" size="small" :columns="columns" :data="dataList"
				@on-selection-change="selectionChange" @on-row-dblclick="edit">
			</Table>
			<div class="demo-spin-col">
				<Spin fix v-if="spinShow">
					<Icon type="ios-loading" size=22 class="demo-spin-icon-load"></Icon>
					<div>正在加载中...</div>
				</Spin>
			</div>
		</div>
		<div class="zs-page-content">
			<div style="float: right;">
				<Page :total="dataCount" :page-size="pageSize" @on-change="changePage"
					@on-page-size-change="changePageSize" show-total show-elevator show-sizer transfer></Page>
			</div>
		</div>
    </div>
</template>

<script>
	import Util from '@/libs/util';
	import zsCommon from '@/libs/common';
	import Cookies from 'js-cookie';
    export default {
        name: '',
        data() {
            return {
				//TODO:配置信息
                moduleURL: undefined, //模块URL（e.g. /zs-erp/zsErpLeave/）
                editPageName: undefined, //编辑页面菜单名（e.g. /leave_edit/）
				//======配置结束==================
                searchContent: undefined,
                spinShow: false,
				columns: [
                    {
						type: 'selection',
						width: 60,
						align: 'center'
					}, {
						type: 'index',
						title: '序号',
						width: 70,
						resizable: true,
						tooltip: true,
						align: 'center',
					},
					{
						title: '编码',
						key: 'code',
						width: 200,
						resizable: true,
						tooltip: true,
						align: 'left',
					},
                    {
						title: '申请人',
						key: 'employeeName',
						width: 200,
						resizable: true,
						tooltip: true,
						align: 'left',
					},
                    {
						title: '所属部门',
						key: 'orgName',
						width: 200,
						resizable: true,
						tooltip: true,
						align: 'left',
					},
					{
						title: '状态',
						key: 'workflowStatus',
						width: 250,
						resizable: true,
						tooltip: true,
						align: 'left',
						render: (h, params) => {
							if (params.row.workflowStatus == "01") {
								return h('span', {
									style: {
										color: "#000000",
									},
								}, '新建')
							}
							if (params.row.workflowStatus == "02") {
								return h('span', {
									style: {
										color: "#aa0000",
									},
								}, params.row.checkPeople)
							}
							if (params.row.workflowStatus == "03") {
								return h('span', {
									style: {
										color: "#aa0000",
									},
								}, '已驳回')
							}
							if (params.row.workflowStatus == "04") {
								return h('span', {
									style: {
										color: "#005500",
									},
								}, '审批完成')
							}
						}
					},
				],
				dataList: [],
				dataCount: 0,
				pageSize: 10,
				pageIndex: 1,
				selectionArr: [],
				searchContent: undefined,
            }
        },
        methods: {
            add() {
                this.$router.push({
					path: this.editPageName + '0'
				});
            },
            del() {
                if (this.selectionArr.length == 0) {
					this.$Message.warning(zsCommon.WARNING_SELECTION_NULL);
					return;
				}
				var ids = [];
				for (var idObj of this.selectionArr) {
					if (idObj.workflowStatus != '01') {
						this.$Message.warning("包含审批的数据，禁止删除");
						return;
					}
					ids.push(idObj.id);
				}
				this.$Modal.confirm({
					title: zsCommon.WARNING_TITLE,
					content: zsCommon.WARNING_DELETE,
					onOk: () => {
						Util.ajax({
							method: 'POST',
							url: this.moduleURL + 'delete',
							data: {
								data: {
									ids: ids
								}
							}
						}).then(res => {
							if (res.data.code == zsCommon.CODE_SUCCESS) {
								this.$Message.info(zsCommon.WARNING_DELETE_SUCCESS);
								this.selectionArr = [];
								this.getData(this.pageIndex, this.pageSize);
							} else {
								this.$Message.error(res.data.msg == undefined ? zsCommon.WARNING_DELETE_ERROR : res.data.msg);
							}
						});
					}
				});
            },
            //获取列表数据
			getData(currentPage, pageSize) {
				//根据url判断是付款还是借款
				let perParams = {};
				if (this.searchContent != undefined && this.searchContent != '') {
					perParams.searchContent = this.searchContent;
				}
				perParams.employeeId = Cookies.get("employeeId");
				this.spinShow = true;
				Util.ajax({
					method: 'POST',
					url: this.moduleURL + 'query',
					data: {
						currentPage: currentPage,
						pageSize: pageSize,
						perParams: perParams
					}
				}).then(res => {
					if (res.data.code == zsCommon.CODE_SUCCESS) {
						this.dataCount = res.data.data.totalNum;
						this.dataList = res.data.data.items;
					}
					this.spinShow = false;
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
            edit(row, index) {
				this.$router.push({
					path: this.editPageName + row.id
				});
			},
        },
        mounted() {
            this.getData(1, this.pageSize);
        },
    }
</script>

<style lang="css" scoped>
    
</style>
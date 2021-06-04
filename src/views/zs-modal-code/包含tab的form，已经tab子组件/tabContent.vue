<template>
    <div id="home-main" class="home-main">
        <div class="zs-btn-content no-bg-btn-content">
			<div class="zs-btn-list" style="padding-left: 0 !important;">
				<div class="btn-compose" @click="beforeAdd" style="padding: 0;margin-left: 0px !important;">
					<img src="../../../../images/newsimg/tianjia.png" style="width: 18px;" />
					<span>新增</span>
				</div>
				<div class="btn-compose" @click="del" style="padding: 0;">
					<img src="../../../../images/newsimg/shanchu.png" style="width: 16px;" />
					<span>删除</span>
				</div>
			</div>
		</div>
		<div style="position: relative;">
			<Table max-height="350" :columns="columns" :data="dataList" @on-selection-change="selectionChange"
				@on-row-dblclick="edit">
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
        <Modal v-model="addModal" title="借用记录编辑" :width="680" :mask-closable="false">
            <Form ref="formItem" class="zs-form-content" :model="formItem" :label-width="120" :rules="validate" style="margin: 0 !important;">
                <Row>
                    <Col span="24">
                    <FormItem label="部门" prop="cancelDate">
                    </FormItem>
                    </Col>
                </Row>
            </Form>
        </Modal>
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
                //==============Table字段=================
                parentId: undefined,
                dataList: [],
				selectionArr: [],
				spinShow: false,
				dataCount: 0,
				pageSize: 5,
				pageIndex: 1,
                columns: [
                    {
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
						title: '备注',
						key: 'remark',
						width: 180,
						resizable: true,
						tooltip: true,
						align: 'left',
					},
				],
                //=============AddModal字段=====================
                addModal: false,
                formItem: {},
                validate: {

                },
            }
        },
        methods: {
            // =====================Table方法=======================
            init(obj) {
				this.parentId = obj.parentId
				this.getData(1, this.pageSize)
			},
            beforeAdd() {
				if (this.parentId == undefined) {
					this.$parent.$parent.$parent.saveBaseInfo("1");
				} else {
					this.add();
				}
			},
			add() {
                this.formItem = {}
                this.$refs.formItem.resetFields()
                this.addModal = true
			},
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
							url: '/zs-erp/zsErpVehicleBorrowRecord/delete',
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
								this.$Message.error(res.data.msg);
							}
						});
					}
				});
			},
			getData(currentPage, pageSize) {
				if (Util.isEmpty(this.parentId)) {
					return
				}
				var perParams = {};
				this.spinShow = true;
				perParams.vehicleId = this.parentId
				Util.ajax({
					method: 'POST',
					url: '/zs-erp/zsErpVehicleBorrowRecord/query',
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
				this.$refs.formItem.resetFields()
                this.formItem = {
                    id: row.id,
					cancelDate: Util.formatDate(row.cancelDate, 'yyyy-MM-dd'),
                }
                this.addModal = true
			},
            //=============addModal方法====================================
            save() {
				this.$refs['formItem'].validate((valid) => {
					if (valid) {
						//新增
						if (this.formItem.id == undefined) {
							//加载中
							Util.loading(this, "保存中...");
							Util.ajax({
								method: 'POST',
								url: '/zs-erp/zsErpVehicleBorrowRecord/add',
								data: {
									data: this.formItem,
								}
							}).then(res => {
								if (res.data.code == zsCommon.CODE_SUCCESS) {
									Util.successMsg(this, zsCommon.INFO_SAVE_SUCCESS);
									this.addModal = false;
									this.getData(1, 5)
								} else {
									Util.errorMsg(this, zsCommon.INFO_SAVE_ERROR);
								}
								//加载结束
								Util.closeLoading(this);
							});
						} else {
							//加载中
							Util.loading(this, "保存中...");
							//更新
							Util.ajax({
								method: 'POST',
								url: '/zs-erp/zsErpVehicleBorrowRecord/update',
								data: {
									data: this.formItem,
								}
							}).then(res => {
								if (res.data.code == zsCommon.CODE_SUCCESS) {
									Util.successMsg(this, zsCommon.INFO_SAVE_SUCCESS);
									this.addModal = false;
									this.getData(1, 5)
								} else {
									Util.errorMsg(this, zsCommon.INFO_SAVE_ERROR);
								}
								//加载结束
								Util.closeLoading(this);
							});
						}
					}
				})
			},
			cancel() {
				this.formItem = {}
				this.$refs.formItem.resetFields()
				this.modal = false
			},
        },
    }
</script>

<style lang="css" scoped>
    
</style>
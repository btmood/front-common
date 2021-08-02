<template>
    <div>
        <div class="zs-btn-content">
			<div class="zs-search-widh">
				<h2 class="zs-type-title">

				</h2>
			</div>
			<!-- 操作按钮区域 -->
			<div class="zs-btn-list" style="padding: 15px 22px;">
				<div class="btn-compose" type="primary" @click="saveBaseInfo">
					<img src="../../../../images/newsimg/baocun.png" style="width: 16px;" />
					<span>保存</span>
				</div>
			</div>
		</div>
        <div class="zs-form-edit">
			<h2 class="zs-form-title">基础信息</h2>
			<div class="zs-form-kuang">
                <Form ref="formItem" class="zs-form-content" :model="formItem" :label-width="120" :rules="ruleValidate"
					style="margin: 0 !important;">
					<Row>
						<Col span="6">
						<FormItem label="申请单编号" prop="code">
							<Input disabled v-model="formItem.code" placeholder="保存后自动生成" type="text"></Input>
						</FormItem>
						</Col>
						<Col span="6">
						<FormItem label="申请人名称" prop="employeeName">
							<Input disabled v-model="formItem.employeeName" type="text"></Input>
						</FormItem>
						</Col>
                        <Col span="6">
						<FormItem label="所属部门" prop="orgName">
							<Input disabled v-model="formItem.orgName" type="text"></Input>
						</FormItem>
						</Col>
						<Col span="6">
						<FormItem label="请假天数" prop="contactType">
							<Input disabled v-model="formItem.leaveDays" type="text"></Input>
						</FormItem>
						</Col>
					</Row>
				</Form>
            </div>
        </div>
        <Tabs type="card">
            <TabPane label="一号tab" name="tabOne">
                <tabContent ref="tabContent"></tabContent>
            </TabPane>
        </Tabs>
    </div>
</template>

<script>
	import Util from '@/libs/util';
	import zsCommon from '@/libs/common';
	import Cookies from 'js-cookie';
    import tabContent from './tabContent.vue'
    export default {
        name: '',
        components: {
            tabContent,
        },
        data() {
            return {
                formItem: {},
                parentInfoId: undefined,
                ruleValidate: {},
            }
        },
        methods: {
            saveBaseInfo(isAddDetail) {
                this.$refs['formItem'].validate((valid) => {
                    if (valid) {
                        //新增
                        if (this.parentInfoId == undefined) {
                            //加载中
                            Util.loading(this, "保存中...");
                            Util.ajax({
                                method: 'POST',
                                url: '/zs-erp/zsErpLeave/add',
                                data: {
                                    data: this.formItem,
                                    fileList: this.uploadList,
                                }
                            }).then(res => {
                                if (res.data.code == zsCommon.CODE_SUCCESS) {
                                    Util.successMsg(this, zsCommon.INFO_SAVE_SUCCESS);
                                    this.parentInfoId = res.data.id; //保存返回主数据的主键ID
                                    this.formItem.code = res.data.leaveCode;
                                    this.$forceUpdate();
                                    //tab初始化
                                    this.$refs.tabContent.init({
										parentId: res.data.id,
									});
									if (isAddDetail == "1") {
										this.$refs.tabContent.add();
									}
                                } else {
                                    Util.errorMsg(this, zsCommon.INFO_SAVE_ERROR);
                                }
                                //加载结束
                                Util.closeLoading(this);
                            });
                        } else {
                            //加载中
                            Util.loading(this, "保存中...");
                            this.formItem.id = this.parentInfoId
                            //更新
                            Util.ajax({
                                method: 'POST',
                                url: '/zs-erp/zsErpLeave/update',
                                data: {
                                    data: this.formItem,
                                    fileList: this.uploadList,
                                }
                            }).then(res => {
                                if (res.data.code == zsCommon.CODE_SUCCESS) {
                                    Util.successMsg(this, zsCommon.INFO_SAVE_SUCCESS);
                                    this.getParams();
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
            getParams() {
                // 取到路由带过来的参数
				if (this.parentInfoId == undefined || this.parentInfoId == 0) {
					this.parentInfoId = this.$route.params.id;
				}
				if (this.parentInfoId == 0) {
					this.parentInfoId = undefined;
				}
				if (this.parentInfoId != undefined) {
					Util.loading(this, "加载中...");
					Util.ajax({
						method: 'POST',
						url: '/zs-erp/zsErpLeave/queryByPrimaryKey',
						data: {
							id: this.parentInfoId,
						}
					}).then(res => {
						if (res.data.code == zsCommon.CODE_SUCCESS) {
							if (res.data.data != undefined) {
                                if (!Util.isEmpty(res.data.data.startTime) && !Util.isEmpty(res.data.data.endTime)) {
                                    this.leaveTime = [Util.formatDate(res.data.data.startTime,"yyyy-MM-dd hh:mm:ss"), Util.formatDate(res.data.data.endTime,"yyyy-MM-dd hh:mm:ss")]
                                }
								this.formItem = res.data.data;
                                //tab初始化
                                this.$refs.tabContent.init({
									parentId: this.parentInfoId,
								});
								this.$refs.tabContent.getData(1, 10);
							}
						}
						Util.closeLoading(this);
					});
				} else {
                    this.formItem.employeeId = Cookies.get("employeeId");
					this.formItem.employeeName = Cookies.get("employeeName");
					this.formItem.orgId = Cookies.get("orgId");
					this.formItem.orgName = Cookies.get("orgName");
					this.formItem.workflowStatus = "01";
					this.$forceUpdate();
                }
            },
        },
        mounted() {
            this.getParams();
        }
    }
</script>

<style lang="css" scoped>
    
</style>
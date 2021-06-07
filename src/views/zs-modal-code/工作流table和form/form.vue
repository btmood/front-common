<template>
    <div>
        <div class="zs-btn-content">
			<div class="zs-search-widh">
				<h2 class="zs-type-title">
                    <span v-if="formItem.workflowStatus == '02' || formItem.workflowStatus == '03'"
						style="font-size: 12px;color: #333333;font-weight: normal;">&nbsp;&nbsp;当前待
                    <span class="editColor">{{checkPeople}}</span> 审批</span>
				</h2>
			</div>
			<!-- 操作按钮区域 -->
			<div class="zs-btn-list" style="padding: 15px 22px;">
				<div class="btn-compose" type="primary" v-if="formItem.workflowStatus=='01'" @click="saveBaseInfo">
					<img src="../../../../images/newsimg/baocun.png" style="width: 16px;" />
					<span>保存</span>
				</div>
                <div class="btn-compose" v-if="formItem.workflowStatus == '01' && parentInfoId != undefined"
					@click="startWorkflow">
					<img src="../../../../images/newsimg/qidong.png" style="width: 16px;" />
					<span>启动</span>
				</div>
                <div class="btn-compose"
					v-if="formItem.workflowStatus != undefined &&  formItem.workflowStatus != '01' && isTaskPage == '1' && taskHasExists"
					@click="doWorkflow('01')">
					<img src="../../../../images/newsimg/shehui.png" style="width: 16px;" />
					<span>驳回</span>
				</div>
                <div class="btn-compose"
					v-if="formItem.workflowStatus != undefined &&  formItem.workflowStatus != '01' && isTaskPage == '1' && taskHasExists"
					@click="doWorkflow('02')">
					<img src="../../../../images/newsimg/tongguo.png" style="width: 16px;" />
					<span>审批</span>
				</div>
				<div class="btn-compose" @click="shenpiClick = true">
					<img src="../../../../images/newsimg/shenpi.png" style="width: 16px;" />
					<span>审批流程</span>
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
					</Row>
                    <Row>
                        <Col span="12">
						<FormItem label="请假事由" prop="reason">
							<Input id="remark" v-model="formItem.reason"
								type="textarea" :autosize="{minRows: 5,maxRows: 5}" />
						</FormItem>
						</Col>
                    </Row>
				</Form>
            </div>
        </div>

        <!-- ================工作流页面========================= -->
		<Modal :width="1200" v-model="shenpiClick" title="审批流程" :mask-closable="false">
			<div style="margin: 10px 22px;">
				<div class="shenpi-title">审批信息</div>
				<Table :columns="workflowLogColumn" :data="approvalList" border style="margin-top:0px"></Table>
				<div style="margin-top: 15px;">
					<div class="shenpi-title">流程图</div>
					<div class="shenpi-img">
						<image-view :images="processImgStr" v-if="processImgStr != undefined" />
						</image-view>
					</div>
				</div>
			</div>
		</Modal>
		<Modal v-model="workflowEditShow" :title="workflowTitle" :width="680" :mask-closable="false">
			<Form style="margin: 16px 22px;" class="zs-form-edit zs-form-content" ref="workflowItem" :model="workflow"
				:label-width="260" :rules="workflowRuleValidate">
				<Row v-if="this.opinionType == '01'">
					<Col span="12">
					<FormItem label="驳回类型">
						<Select v-model="rollbackType" style="width: 260px !important;">
							<!-- <Option value="1" id="1" key="1">结束流程</Option> -->
							<Option value="2" id="2" key="2">驳回至发起人</Option>
							<Option value="3" id="3" key="3">驳回至其它审批人</Option>
						</Select>
					</FormItem>
					</Col>
				</Row>
				<Row>
					<Col span="10">
					<FormItem label="驳回节点" v-if="rollbackType == '3'" style="width: 320px;">
						<Select v-model="rollbackNodeId" style="width: 260px !important;">
							<Option v-for="item in historicNodeList" :key="item.id" :value="item.id">{{item.name}}
							</Option>
						</Select>
					</FormItem>
					</Col>
				</Row>
				<Row>
					<Col span="24">
					<FormItem label="审批意见" prop="suggest">
						<Input v-model="workflow.suggest" type="textarea" :autosize="{minRows: 2,maxRows: 2}"
							placeholder="审批意见"></Input>
					</FormItem>
					</Col>
				</Row>
			</Form>
			<div slot="footer">
				<Button type="primary" @click="saveWorkflow">确认</Button>
			</div>
		</Modal>
    </div>
</template>

<script>
	import Util from '@/libs/util';
	import zsCommon from '@/libs/common';
	import Cookies from 'js-cookie';
    import workflowCommon from '@/views/pages/zsplat-zero/zsplat-workflow/common/workflow';
    import imageView from '@/views/main-components/image-view';
    export default {
        components: {
            imageView
        },
        name: '',
        data() {
            return {
                //TODO:配置信息
                moduleURL: undefined, //模块URL（e.g. /zs-erp/zsErpLeave/）
                tablePageName: undefined, //table页面菜单名（e.g. /leave）
                tableId: undefined, //表唯一标识ID，后端controller中最上面的变量
                //=================表单字段=====================
                formItem: {},
                parentInfoId: undefined,
                ruleValidate: {
					reason: [{
						required: true,
						type: "string",
						message: '请假事由为空',
						trigger: 'blur'
					}],
				},
                //==================工作流字段====================
                taskHasExists: false, //判断该任务是否存在
				taskHasSuspended: false, //判断任务状态激活、挂起
				startNode: {}, //流程开始节点
				historicNodeList: [], //历史节点集合
				checkPeople: '', //当前审批人
				taskId: undefined, //当前任务ID
				taskName: undefined, //当前结点名称
				approvalList: [], //审批记录
				processImgStr: undefined, //流程图地址
				rollbackType: '2', //驳回类型
				workflowTitle: '', //审批模态框
				opinionType: "01", //意见类型
				workflowEditShow: false, //是否显示流程审批
				workflow: {
					suggest: ''
				}, //工作流
				isTaskPage: '0', //控制是否显示工作流操作按钮
				shenpiClick: false,
				workflowLogColumn: [{
						title: '节点名称',
						key: 'actname',
						align: 'left',
						width: 200,
						render: (h, params) => {
							return h('div', {
								style: {
									'textAlign': 'left'
								}
							}, params.row.actname);
						}
					},
					{
						title: '审批人',
						key: 'executorname',
						align: 'left',
						width: 100
					},
					{
						title: '审批时间',
						key: 'createTime',
						sortable: true,
						align: 'left',
						width: 200,
						render: (h, params) => {
							return h('span', Util.formatDate(params.row.createTime, 'yyyy-MM-dd hh:mm:ss'));
						}
					},
					{
						title: '处理时长(小时)',
						align: 'left',
						sortable: true,
						width: 180,
						render: (h, params) => {
							let showObj = params.row;
							let showContent = undefined;
							if (showObj != undefined) {
								showContent = showObj.duration;
							}
							return h('span', (showContent / 3600000).toFixed(2));
						}
					},
					{
						title: '审批结果',
						key: 'opiniontype',
						align: 'left',
						width: 100,
						render: (h, params) => {
							return h('span', workflowCommon.judgeOpinionType(params.row.opiniontype));
						}
					},
					{
						title: '审批意见',
						key: 'opinion',
						align: 'left',
						render: (h, params) => {
							return h('div', {
								style: {
									'textAlign': 'left'
								}
							}, params.row.opinion);
						}
					}
				],
				workflowRuleValidate: {
					suggest: [{
						type: 'string',
						required: true,
						message: '审批意见为空',
						trigger: 'blur'
					}]
				},
				rollbackNodeId: undefined, //驳回指定节点ID
            }
        },
        methods: {
            //========================表单方法===========================
            saveBaseInfo(isAddDetail, isStartWorkflow) {
                this.$refs['formItem'].validate((valid) => {
                    if (valid) {
                        //新增
                        if (this.parentInfoId == undefined) {
                            //加载中
                            Util.loading(this, "保存中...");
                            Util.ajax({
                                method: 'POST',
                                url: moduleURL + 'add', //拼接URL
                                data: {
                                    data: this.formItem,
                                    fileList: this.uploadList,
                                }
                            }).then(res => {
                                if (res.data.code == zsCommon.CODE_SUCCESS) {
                                    if (isStartWorkflow != true) {
                                        Util.successMsg(this, zsCommon.INFO_SAVE_SUCCESS);
                                    } else {
                                        this.actionStart();
                                    }
                                    this.parentInfoId = res.data.id; //保存返回主数据的主键ID
                                    this.formItem.code = res.data.code;
                                    this.$forceUpdate();
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
                                url: moduleURL + 'update', //拼接URL
                                data: {
                                    data: this.formItem,
                                    fileList: this.uploadList,
                                }
                            }).then(res => {
                                if (res.data.code == zsCommon.CODE_SUCCESS) {
                                    if (isStartWorkflow != true) {
                                        Util.successMsg(this, zsCommon.INFO_SAVE_SUCCESS);
                                    } else {
                                        this.actionStart();
                                    }
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
                let query = this.$route.params.query;
				if (query != undefined) {
					this.isTaskPage = query.taskFlag;
					if (this.isTaskPage == '1') {
						this.taskId = query.taskId;
						this.taskName = query.nodeName;
						this.nodeId = query.nodeId;
						this.programId = query.programId;
						this.opers = query.opers;
						//判断任务是否存在
						this.checkTaskHasExists();
					}
				}
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
						url: moduleURL + 'queryByPrimaryKey', //拼接URL
						data: {
							id: this.parentInfoId,
						}
					}).then(res => {
						if (res.data.code == zsCommon.CODE_SUCCESS) {
							if (res.data.data != undefined) {
								this.formItem = res.data.data;
                                this.queryUserTaskNodeList();
                                this.getCheckPeople()
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
            //======================工作流方法==============================
            //查询任务是否存在
			checkTaskHasExists() {
				Util.ajax({
					method: "POST",
					url: "/workflow/workflowBase/taskHasExists",
					data: {
						taskId: this.taskId,
					}
				}).then(res => {
					if (res.data.code == zsCommon.CODE_SUCCESS) {
						this.taskHasExists = res.data.hasExists;
						if (this.taskHasExists) { //任务存在
							this.taskHasSuspended = res.data.hasSuspended; //任务状态 false:挂起
						}
					}
				});
			},
            startWorkflow() {
                this.$Modal.confirm({
					title: zsCommon.WARNING_TITLE,
					content: zsCommon.WARNING_START,
					onOk: () => {
						//启动前先保存
						this.saveBaseInfo(false, true);
					}
				});
            },
            actionStart() {
				this.$Spin.show();
				Util.ajax({
					method: 'POST',
					url: moduleURL + 'startWorkflow', //拼接URL
					data: {
						programeId: this.parentInfoId,
					}
				}).then(res => {
					this.$Spin.hide();
					if (res.data.code == zsCommon.CODE_SUCCESS) {
						this.$Message.info(zsCommon.INFO_START_SUCCESS);
						this.queryUserTaskNodeList();
						//加载审批表
						this.initApprovalList();
						//加载流程图
						this.initWorkflowImg();
						this.$set(this.formItem, "workflowStatus", "02"); //状态设置为审批中
						//跳转到列表页面
						this.back()
					} else {
						if (res.data.msg != undefined) {
							this.$Message.error(res.data.msg);
						} else {
							this.$Message.error(zsCommon.INFO_START_ERROR);
						}
					}
				});
			},
            back() {
				this.$router.push({
					path: tablePageName //table菜单名
				});
			},
            //查询历史节点信息
			queryUserTaskNodeList() {
				if (this.parentInfoId != undefined) {
					Util.ajax({
						method: "POST",
						url: "/workflow/workflowBase/getHistoricNodeList",
						data: {
							bussinessKey: this.parentInfoId,
						}
					}).then(res => {
						if (res.data.code == zsCommon.CODE_SUCCESS) {
							let nodeList = res.data.data;
							this.startNode = nodeList[0];
							//第一个节点和最后一个节点去除
							//第一个为开始节点，与驳回至发起人功能一样
							//最后一个节点为当前节点，无意义
							let lastNodeList = [];
							for (let i = 0; i < nodeList.length; i++) {
								if (i != 0 && i != (nodeList.length - 1)) {
									lastNodeList.push(nodeList[i]);
								}
							}
							this.historicNodeList = lastNodeList;
						}
					});
				}
			},
            //加载审批表
			initApprovalList() {
				let taskid = '';
				if (this.parentInfoId != undefined) {
					taskid = this.parentInfoId
				} else if (this.taskid != undefined) {
					taskid = this.taskid
				} else {
					return
				}
				//加载审批表
				Util.ajax({
					method: 'POST',
					url: '/workflow/zsSysWorkflowApproval/query',
					data: {
						currentPage: null,
						pageSize: null,
						perParams: {
							taskid: taskid
						}
					}
				}).then(res => {
					if (res.data.code == zsCommon.CODE_SUCCESS) {
						this.approvalList = res.data.data.items;
					}
				});
			},
            //加载流程图
			initWorkflowImg() {
				this.processImgStr = undefined;
				let pragrameId = '';
				if (this.parentInfoId != undefined) {
					pragrameId = this.parentInfoId
				} else if (this.taskid != undefined) {
					pragrameId = this.taskid
				} else {
					this.myInitWorkflowImg()
					return
				}
				Util.ajax.get("/workflow/workflowBase/queryWorkflowImg?pragrameId=" + pragrameId, {
					responseType: 'arraybuffer',
				}).then(res => {
					return 'data:image/png;base64,' + btoa(new Uint8Array(res.data)
						.reduce((data, byte) => data + String.fromCharCode(byte), '')
					);
				}).then(data => {
					let a = data.length
					if (a > 200) {
						this.processImgStr = data;
					} else {
						this.myInitWorkflowImg()
					}
				});
			},
			myInitWorkflowImg() {
				Util.ajax.get(
					"/workflow/workflowBase/queryWorkflowImgByTableId?tableId=" + tableId, {
						responseType: 'arraybuffer',
					}).then(res => {
					return 'data:image/png;base64,' + btoa(new Uint8Array(res.data)
						.reduce((data, byte) => data + String.fromCharCode(byte), '')
					);
				}).then(data => {
					this.processImgStr = data;
				});
			},
            getCheckPeople() {
				if (this.formItem.workflowStatus != "02" && this.formItem.workflowStatus != "03") {
					return
				}
				Util.ajax({
					method: "POST",
					url: "/workflow/workflowBase/getCurrentNodeInfo",
					data: {
						programeId: this.parentInfoId,
					}
				}).then(res => {
					let node = res.data.data.actName;
					node = node.replace(/审批/g, "");
					this.checkPeople = "【" + node + "】" + res.data.data.nodeAssigneeEmployeeNames;
				});
			},
            doWorkflow(name) {
                this.rollbackType = '2'; //默认驳回至发起人
				switch (name) {
					//驳回
					case workflowCommon.ROLLBACK:
						this.workflowTitle = "【驳回】审批";
						this.opinionType = workflowCommon.ROLLBACK;
						this.workflowEditShow = true;
						this.workflow.suggest = '';
						break;
						//提交
					case workflowCommon.COMPLETE:
						this.workflowTitle = "【提交】审批";
						this.opinionType = workflowCommon.COMPLETE;
						this.workflowEditShow = true;
						this.workflow.suggest = '同意';
						break;
				}
            },
            saveWorkflow() {
				this.$refs['workflowItem'].validate((valid) => {
					if (valid) {
						switch (this.opinionType) {
							//驳回
							case workflowCommon.ROLLBACK:
								this.rollback();
								break;
								//提交
							case workflowCommon.COMPLETE:
								this.complete();
								break;
						}
					}
				});
			},
			rollback() {
				let params = {
					taskId: this.taskId,
					bussinessId: this.parentInfoId,
					opintion: this.workflow.suggest,
					rollbackType: this.rollbackType
				};
				//驳回至发起人
				if (this.rollbackType == '2') {
					if (this.startNode.id == undefined) {
						this.$Message.warning("暂无开始节点");
						return;
					}
					params.rollbackNodeId = this.startNode.id;
				}
				//驳回至指定其它审批人
				if (this.rollbackType == '3') {
					if (this.rollbackNodeId == undefined) {
						this.$Message.warning("请选择驳回节点");
						return;
					}
					params.rollbackNodeId = this.rollbackNodeId;
				}
				this.$Spin.show();
				Util.ajax({
					method: 'POST',
					url: moduleURL + 'rollback', //拼接URL
					data: params
				}).then(res => {
					this.$Spin.hide();
					if (res.data.code == zsCommon.CODE_SUCCESS) {
						this.$Message.info("驳回成功");
						this.$router.push({
							path: '/home'
						});
						// this.getParams();
						// //加载审批表
						// this.initApprovalList();
						// //加载流程图
						// this.initWorkflowImg();
						// //判断任务是否存在
						// this.checkTaskHasExists();
					} else {
						this.$Message.error("驳回失败");
					}
					this.workflowEditShow = false;
				});
			},
			complete() {
				let data = {
					taskId: this.taskId,
					taskName: this.taskName,
					opintion: this.workflow.suggest,
					opinionType: '01',
					bussinessId: this.parentInfoId,
				}
				this.$Spin.show();
				Util.ajax({
					method: 'POST',
					url: moduleURL + 'completeWorkflow', //拼接URL
					data: data
				}).then(res => {
					this.$Spin.hide();
					if (res.data.code == zsCommon.CODE_SUCCESS) {
						this.$Message.info("审批成功");
						this.getParams();
						//加载审批表
						this.initApprovalList();
						//加载流程图
						this.initWorkflowImg();
						//判断任务是否存在
						this.checkTaskHasExists();
					} else {
						this.$Message.error("请勿重复提交审批");
					}
					this.workflowEditShow = false;
				});
			},
        },
        mounted() {
            this.getParams();
            //加载审批表
			this.initApprovalList();
			//加载流程图
			this.initWorkflowImg();
        }
    }
</script>

<style lang="css" scoped>
</style>
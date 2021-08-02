<template>
    <div>
        <!-- 各种form表单组件 -->
        <Form ref="formItem" class="zs-form-content" :model="formItem" :label-width="120" :rules="ruleValidate" style="margin: 0 !important;">
            <Row>
                <!-- input text -->
                <Col span="6">
                    <FormItem label="申请单编号" prop="code">
                        <Input v-model="formItem.code" placeholder="保存后自动生成" type="text"></Input>
                    </FormItem>
                </Col>
                <!-- input 金额 -->
                <Col span="6">
                    <FormItem label="预支借款" prop="borrowMoney">
                        <InputNumber :min="0"
                            :formatter="value => `￥ ${value}`.replace(/\B(?=(\d{3})+(?!\d))/g, ',')"
                            :parser="value => value.replace(/\￥\s?|(,*)/g, '')" v-model="formItem.borrowMoney">
                        </InputNumber>
                    </FormItem>
                </Col>
                <!-- select v-for -->
                <Col span="6">
                    <FormItem label="请假类型" prop="type" class="ding-search" style="position: relative;">
                        <Select filterable v-model="formItem.type">
                            <Option v-for="item in leaveType" :value="item.id" :key="item.id" :label="item.value">
                            </Option>
                        </Select>
                    </FormItem>
                </Col>
                <!-- select 自己定义item -->
                <Col span="12">
                    <FormItem label="驳回类型" prop="rollbackType">
                        <Select v-model="formItem.rollbackType" style="width: 260px !important;">
                            <Option value="1">结束流程</Option>
                            <Option value="2">驳回至发起人</Option>
                            <Option value="3">驳回至其它审批人</Option>
                        </Select>
                    </FormItem>
                </Col>
                <!-- 级联 -->
                <Col span="6">
                    <FormItem label="所属项目" prop="clientProjectId">
                        <Cascader v-model="formItem.clientProjectId" :data="projectClientList">
                        </Cascader>
                    </FormItem>
                </Col>
                <!-- 单个时间 -->
                <Col span="6">
                    <FormItem label="登记日期" prop="registrationDate">
                        <DatePicker v-model="formItem.registrationDate" type="date" format="yyyy-MM-dd" placement="bottom-end" placeholder="请选择时间段" transper></DatePicker>
                    </FormItem>
                </Col>
                <!-- 时间段 -->
                <Col span="6">
                    <FormItem label="请假时间" prop="leaveTime">
                        <DatePicker class="selectTime" v-model="formItem.leaveTime" @on-change="dateRangeChange" type="datetimerange" format="yyyy-MM-dd HH:mm" placement="bottom-start" placeholder="请选择时间段" transper></DatePicker>
                    </FormItem>
                </Col>
            </Row>
            <Row>
                <!-- 大文本框 -->
                <Col span="12">
                <FormItem label="备注" prop="remark">
                    <Input id="remark" v-model="formItem.remark"
                        type="textarea" :autosize="{minRows: 4,maxRows: 4}" />
                </FormItem>
                </Col>
            </Row>
        </Form>
    </div>
</template>

<script>
    export default {
        name: '',
        data() {
            return {
                formItem: {},
                leaveType: [],
                projectClientList: {},
                ruleValidate: {
                    type: [{
						required: true,
						type: "string",
						message: '请假类型为空',
						trigger: 'blur'
					}],
					leaveTime: [{
						required: true,
						type: "date",
						trigger: 'blur',
                        validator: (rule, value, callback) => {
                            if (Util.isEmpty(this.formItem.startTime) || Util.isEmpty(this.formItem.endTime)) {
                                callback(new Error('请假时间为空'))
                            }
                            callback()
                        },
					}],
                }
            }
        },
        methods: {
            // 时间段改变
            dateRangeChange(date){
                if(date[0] == "" || date[1] == ""){
                    this.formItem.startTime = null;
                    this.formItem.endTime = null;
                    return;
                }
                this.formItem.startTime = Util.formatDate(date[0],"yyyy-MM-dd hh:mm:ss");
                this.formItem.endTime = Util.formatDate(date[1],"yyyy-MM-dd hh:mm:ss");
            },
        }
    }
</script>

<style lang="" scoped>
    
</style>
<template>
    <div>
        <Modal :width="460" v-model="modal" title="标题" :mask-closable="false" class="zs-form-content">
			<Form class="zs-form-content" ref="formItem" :model="formItem" :label-width="100" :rules="ruleValidate"
				style="margin-top: 16px;">
					<Col span="24" class="textareaStyle">
					<FormItem label="经办人" prop="employeeId">
						<Input disabled v-model="formItem.employeeName" />
					</FormItem>
					</Col>
				</Row>
			</Form>
			<div slot="footer">
				<Button type="primary" @click="save">保存</Button>
				<Button @click="cancel">取消</Button>
			</div>
		</Modal>
    </div>
</template>

<script>
    export default {
        name: '',
        data() {
            return {
                modal: false,
                formItem: {},
                ruleValidate: {},
            }
        },
        methods: {
            save() {},
            cancel() {},
        }
    }
</script>

<style lang="css" scoped>
    
</style>
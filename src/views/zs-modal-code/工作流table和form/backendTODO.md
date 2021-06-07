
## 1待修改方法

### 1.1 `startWorkflow()`

需要添加更新工作流状态的代码

```
public Map<String,Object> startWorkflow(@RequestBody Map<String,Object> params,Authentication authentication) {
        if (judgeBlank(params.get("programeId"))) {
            return baseResultMap;
        }
        Map<String,Object> resultMap = new HashMap<String,Object>();
        String programeId = params.get("programeId").toString();

        try {
            ZsRbacUser user = getCurrentUser(authentication);
            Map<String,Object> paramsMap = new HashMap<String,Object>();
            paramsMap.put("isLongVacation", params.get("isLongVacation"));
            resultMap = start(this.TABLE_ID,programeId,user,paramsMap);
            if (ConstantUtil.CODE_SUCCESS.equals(resultMap.get("code").toString())) {
                //更新流程状态
                Map<String, Object> pa = new HashMap<String, Object>();
                pa.put("id", programeId);
                pa.put("workflowStatus", WorkflowConstants.WorkflowStatus.DOING.getValue());//审批中
                zsErpLeaveService.updateById(pa);
            } else {
                return resultMap;
            }
        } catch (Exception e) {
            e.printStackTrace();
            resultMap.put("code", ConstantUtil.CODE_ERROR);
            logger.error(ConstantUtil.CODE_ERROR, e);
            return resultMap;
        }
        resultMap.put("code",ConstantUtil.CODE_SUCCESS);
        return resultMap;
    }
```

### 1.2 `rollback()`

工作流驳回需要添加更新表状态代码

```
public Map<String,Object> workflowRollback(@RequestBody Map<String,Object> params,Authentication authentication) {
        Map<String,Object> resultMap = new HashMap<String,Object>();
        if (judgeBlank(params.get("taskId"))) {
            return baseResultMap;
        }
        if (judgeBlank(params.get("bussinessId"))) {
            return baseResultMap;
        }
        if (judgeBlank(params.get("opintion"))) {
            return baseResultMap;
        }
        if (judgeBlank(params.get("rollbackType"))) {
            return baseResultMap;
        }
        String taskId = params.get("taskId").toString();//任务ID
        String bussinessId = params.get("bussinessId").toString();//业务数据ID
        String opintion = params.get("opintion").toString();//意见内容
        String rollbackType = params.get("rollbackType").toString();//驳回类型

        Map<String,Object> pa = new HashMap<String,Object>();
        //直接中止流程
        if ("1".equals(rollbackType)) {
            endWorkflow(taskId, bussinessId, opintion, authentication);
            //更新流程状态
            pa.put("id", bussinessId);
            pa.put("workflowStatus", WorkflowConstants.WorkflowStatus.BACK.getValue());//审批驳回
            zsErpLeaveService.updateById(pa);
        } else if ("2".equals(rollbackType)) {
            //驳回至发起人
            if (judgeBlank(params.get("rollbackNodeId"))) {
                return baseResultMap;
            }
            //驳回指定节点
            String rollbackNodeId = params.get("rollbackNodeId").toString();
            rollBack(taskId, bussinessId, rollbackNodeId, opintion, true, authentication);
            //更新流程状态
            pa.put("id", bussinessId);
            pa.put("workflowStatus", WorkflowConstants.WorkflowStatus.EDIT.getValue());//驳回编辑状态
            zsErpLeaveService.updateById(pa);
        } else {
            //驳回至指定节点
            if (judgeBlank(params.get("rollbackNodeId"))) {
                return baseResultMap;
            }
            //驳回指定节点
            String rollbackNodeId = params.get("rollbackNodeId").toString();
            rollBack(taskId, bussinessId, rollbackNodeId, opintion, false, authentication);
            //更新流程状态
            pa.put("id", bussinessId);
            pa.put("workflowStatus", WorkflowConstants.WorkflowStatus.BACK.getValue());//审批驳回
            zsErpLeaveService.updateById(pa);
        }
        resultMap.put("code",ConstantUtil.CODE_SUCCESS);
        return resultMap;
    }
```

### 1.3`completeWorkflow()`

更新工作流状态需要添加更新表状态代码

```
public Map<String,Object> completeWorkflow(@RequestBody Map<String,Object> params,Authentication authentication) {
        Map<String,Object> resultMap = new HashMap<String,Object>();
        if (judgeBlank(params.get("taskId"))) {
            return baseResultMap;
        }
        if (judgeBlank(params.get("bussinessId"))) {
            return baseResultMap;
        }
        if (judgeBlank(params.get("opintion"))) {
            return baseResultMap;
        }
        String taskId = params.get("taskId").toString();//待办任务ID
        String bussinessId = params.get("bussinessId").toString();//业务ID
        String opintion = params.get("opintion").toString();//意见内容
        try {
            ZsRbacUser user = getCurrentUser(authentication);
            Map<String,Object> paramsMap = new HashMap<String,Object>();
            complete(taskId,user,WorkflowConstants.WorkflowOpinionType.COMPLETE.getValue(),opintion,paramsMap);
            //判断流程是否还存在，没有说明审批结束
            Map<String, Object> pa = new HashMap<String, Object>();
            pa.put("id", bussinessId);
            if (!judgeHasExist(bussinessId)) {
                //更新流程状态
                pa.put("workflowStatus", WorkflowConstants.WorkflowStatus.SUCCESS.getValue());//审批结束
                initUpdateParams(pa, authentication);
                zsErpLeaveService.updateById(pa);
            } else {
                pa.put("workflowStatus", WorkflowConstants.WorkflowStatus.DOING.getValue());//审批同意
                initUpdateParams(pa, authentication);
                zsErpLeaveService.updateById(pa);
            }
        } catch (Exception e) {
            resultMap.put("code", ConstantUtil.CODE_ERROR);
            logger.error(ConstantUtil.CODE_ERROR, e);
            return resultMap;
        }
        resultMap.put("code",ConstantUtil.CODE_SUCCESS);
        return resultMap;
    }
```
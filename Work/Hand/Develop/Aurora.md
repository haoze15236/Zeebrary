# Aurora

官方API：[http://aurora.hand-china.com/](http://aurora.hand-china.com/)

## 前端

### 发送Ajax请求

```javascript
var headerDs = $au('dataSet的ID');
//验证是否有修改
if (!$au('dataSet的ID').isModified()) {
    $au('dataSet的ID').data[0].dirty = true;
}
//校验必输
if (!$au('dataSet的ID').validate()) {
    return;
}
//验证是否选了某一行数据
if (records.length < 1) {
    Aurora.showInfoMessage('$[l:prompt]','$[l:prompt.please_select]');
    return;
}
//构造参数，数据格式：[{…}, {…}]
//从dataSet获取选中数据
var param = headerDs.getJsonData(true); 
//自行拼接参数
var record = $au('dataSet的ID').getAt(0);
var param = {};
param.paymentDate = record.get('paymentDate');
param.currencyCode = record.get('currencyCode');
param.exchangeRateType = record.get('exchangeRateType');
param.exchangeRate = record.get('exchangeRate');
param.bankAccountId = record.get('bankAccountId');
param.cashFlowItemId = record.get('cashFlowItemId');
param.paymentMethodId = record.get('paymentMethodId');
param.description = record.get('description');
param.data = $au('CSH5040_docPaymentResultDs').getJsonData(true);
param = [param];
//发送Ajax请求
Aurora.request({
    lockMessage: '$[l:hap_waiting]',
    url: $au('CSH5040_executePaymentLink').getUrl(),
    para: param,
    sync:true,
    success: function (resp) {
        Aurora.Masker.unmask($au('CSH5040_csh_document_payment_pay_window').wrap);
        CSH5040_returnPayment();
    },
    failure: function () {
        Aurora.Masker.unmask($au('CSH5040_csh_document_payment_pay_window').wrap);
    },
    error: function () {
        Aurora.Masker.unmask($au('CSH5040_csh_document_payment_pay_window').wrap);
    },
    scope: this
});
```

### 锁屏

```javascript
Aurora.Masker.mask($au('window的ID').wrap,'$[l:hap.pay_waiting]');
Aurora.Masker.unmask($au('window的ID').wrap);
//新开页面
 new Aurora.Window({
                id: 'activity_create_window',
                url: $au('pm_apr_activity_create_modify_link').getUrl() + "?type=add",
                title: '$[l:pm_apr_activity.activity_new]',
                side: 'right',
                fullScreen: true,
                width: 1020
            }).on('close', function() {
                <!--取消页面关闭触发的查询-->
                $au('pm_apr_activity_result_ds').query();
            });
```

### 页面嵌套校验必输

```javascript
//审批同意自动保存
if (Aurora.CmpManager.get('commentsdata')) {
    var func = approveFun;
    var approveFun = function(actionId, actionType) {
        if (actionType == '1') {
            //校验必输,调用保存方法
            savereport();
            $('exp_report_head')['on']('submitsuccess', function() {
                func(actionId, actionType);
            })
        } else {
            func(actionId, actionType);
        }
    }
}
```

### TAG属性

```javascript
//显示时分秒
renderer="Aurora.formatDateTime"
//光标定位到当前组件，如果有内容会全选内容
$au('ID').focus();
$au('id').wrap.child('input[atype=field.input]');
$au('id').onFocus;

```

### 列汇总

```
        function CSH3200_summaryRenderer(datas, name) {
            var sum = 0;
            for (var i = 0;i < datas.length;i++) {
                var r = datas[i];
                var d = r.get(name);
                var n = parseFloat(d);
                if (!isNaN(n)) {
                    sum += n;
                }
            }
            return '<font>' + Aurora.formatMoney(sum, 0) + '</font>';
        }

```

### 弹框填描述信息

```javascript
//外部页面
function CSH5040_fundsDoc(){
            var headerDs = $au('CSH5040_docPaymentResultDs');
            var records = headerDs.getJsonData(true);
            //验证是否选了某一行数据
            if (records.length < 1) {
                Aurora.showInfoMessage('$[l:prompt]', '$[l:prompt.please_select]');
                return;
            }
            //部分支付的记录不允许资金撤回
            for(var i = 0 ; i < records.length; i++){
                var payedAmount = records[i].payedAmount;
                if(payedAmount !== 0){
                    Aurora.showInfoMessage('$[l:prompt]', '$[l:csh5010_funds_records_error]');
                    return;
                }
            }
            new Aurora.Window({
                url: $au('CSH5040_fundspay_link').getUrl(),
                id: 'CSH5040_fundspay_window',
                title: '$[l:hap_operation]',
                width: 500,
                height: 250
            });
        }
//弹框页面
<?xml version="1.0" encoding="UTF-8"?>
<a:screen xmlns:a="http://www.aurora-framework.org/application">
    <a:view>
        <script><![CDATA[
            function CSH5040_doProcess() {
                var recordsData = $au('CSH5040_docPaymentResultDs').getJsonData(true);
                var opinion = $au('CSH5040_process_ds').getAt(0).get('opinion');
                if (!$au('CSH5040_process_ds').validate()) {
                    return;
                }
                for (var i = 0;i < recordsData.length;i++) {
                    recordsData[i]['opinion'] = opinion;
                }
                Aurora.request({
                    lockMessage: '$[l:hap_waiting]',
                    url: $au('CSH5040_doc_funds_link').getUrl(),
                    para: recordsData,
                    success: function () {
                        $au('CSH5040_fundspay_window').close();
                        $au('CSH5040_docPaymentResultDs').query();
                    },
                    scope: this
                });
            }
            
            function CSH5040_cancelProcess() {
                $au('CSH5040_fundspay_window').close();
            }
        ]]></script>
        <a:dataSets>
            <a:dataSet id="CSH5040_process_ds" autoCreate="true">
                <a:fields>
                    <a:field name="opinion" required="true"/>
                </a:fields>
            </a:dataSet>
        </a:dataSets>
        <a:form padding="0" showmargin="false" shrinkable="false" style="border:none;" title="ptl_logs.log_view">
            <a:formToolbar>
                <a:label name="separator"/>
                <a:button click="CSH5040_doProcess" text="confirm" width="80"/>
                <a:button click="CSH5040_cancelProcess" text="hap.cancel" width="80"/>
            </a:formToolbar>
            <a:textArea name="opinion" bindTarget="CSH5040_process_ds" height="200" width="490"/>
        </a:form>
    </a:view>
</a:screen>

```



# 开发规范

前端组件ID以文件路径开头，加文件名+组件类型：`ssc_doc_csh_payment_audit_finance_grid`
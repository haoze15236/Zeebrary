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
```


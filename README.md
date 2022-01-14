**初衷：一直以来，表单校验就是一个常用的功能，但是我做了几个月的小程序，却没找到让我心满意足的表单校验组件，因此决定自己撸一个出来。**

下载地址：[wx-formValidate](https://github.com/zhouxni/wx-formValidate)

**基础用法：**

引入form和field组件，同时要保持与instance.js同级目录。(注意：由于小程序的诸多限制，如果需要修改相关的css样式和布局，可以进入到组件的wxml和wxss修改。)

<br/>

### 用法

**Form**

**props**

|参数|说明对象|类型|默认值|
|--|--|--|--|
|model|表单数据对象|object||
|rules|表单验证规则|object||

<br/>

**事件**

|事件名称|说明|回调参数|
|--|--|--|
|bind:validate|任一表单项被校验后触发|e.detail:被校验的表单项 prop 值，校验是否通过，错误消息|
|bind:submit|数据验证成功后回调事件|-|

<br/>

**方法**

|方法名|说明|参数|
|--|:--|--|
|validate|对整个表单进行校验的方法，参数为一个回调函数。该回调函数会在校验结束后被调用，并传入两个参数：是否校验成功和未通过校验的字段信息。|Function(callback: Function(boolean, array))|
|validateField|对部分表单字段进行校验的方法，返回每个字段是否校验成功状态和相应字段及信息|Function(props: array \| string, callback: Function(errorMessage: object|array))|
|resetFields|对整个表单进行重置，将所有字段值重置为初始值并移除校验结果|-|
|clearValidate|移除表单项的校验结果。传入待移除的表单项的 prop 属性或者 prop 组成的数组。|Function(props: array \| string)|

注意；使用this.selectComponent('#form')获取form组件实例后，就可以调用这些方法。('#form'为form组件绑定的id)

```
<form model="{{model}}" rules="{{rules}}" id="form">
...
</form>
```

```
    this.setData({
      form: this.selectComponent('#form')
    })
    ////
    this.data.form.resetFields()
```

**Field**

默认是个输入框组件，可以使用custom开启slot，插入自定义表单组件。

**props**

|参数|说明|类型|默认值|
|--|--|--|--|
|prop|表单域 model对应的 字段|string|-|
|value|表单域的值|不限|-|
|label|label文本|string|-|
|placeholder|文本占位符|string|'请输入'|
|placeholderStyle|文本占位符自定义样式|string|-|
|custom|是否插入自定义表单组件|boolean|false|
|autoChecked|对于其它非input组件，对值进行自动校验|boolean|true|

<br/>

**事件**

|事件名|说明|回调参数|
|--|--|--|
|bind:change|input输入框值改变时触发|e.detail:{prop:字段名,value:变化后的值}|
|bind:blur|input离开焦点后触发|-|

**校验规则**

本插件为了简化，基本没有内置校验规则，都要开发者手动设置，规则在rules中配置。

示例如下：

```
    rules: {
      name: [{
        validator: (value, callback) => {
          if (!value) {
            callback(new Error('请输入姓名'))
          }
        },
        trigger:'blur'
      }],
      phone: [{
        validator: (value, callback) => {
          if (!value) {
            callback(new Error('请输入手机号'))
          }
        }
      }],
      checked: [{
        validator: (value, callback) => {
          if (!value) {
            callback('请打开开关')
          }
        }
      }]
    }
```

每个字段可以配置多个规则，每个规则对象有validator和trigger属性，validator为规则函数，第一个参数是字段值，第二个是抛出相应错误信息的函数，参数为string或error实例(一旦捕捉到错误信息，同一字段的后面规则便不再执行)。trigger为触发条件，可以为字符串或者数组，值为blur' | 'change' | ['change', 'blur']，默认为['change', 'blur']。基本是input组件使用trigger这个属性。其它非input组件可以使用autoChecked进行自动实时校验，类似‘change’。或者使用form中validateField方法在相关事件函数中手动校验。

**template模板代码**

```
<form model="{{model}}" rules="{{rules}}" id="form">
  <field prop="name" value="{{model.name}}" placeholder="请输入姓名" label="姓名" bind:change="onChange" />
  <field prop="phone" value="{{model.phone}}" placeholder="请输入手机号" label="手机" bind:change="onChange" />
  <field custom prop="checked" value="{{model.checked }}" label="开关">
    <van-switch checked="{{ model.checked }}" bind:change="onChangec" />
  </field>
</form>
```

```
model:{
      name: '',
      phone: '',
      checked: true
  },
  /////
 onChange(e) {
    const {
      prop,
      value
    } = e.detail
    this.data.model[prop] = value
    this.setData({
      model: this.data.model
    })
  },
  onChangec({
    detail
  }) {
    this.data.model['checked'] = detail
    this.setData({
      model: this.data.model
    })
  },
```

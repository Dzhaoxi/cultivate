# mui入坑笔记

在mui app开发中，若要使用[HTML5+扩展api](https://www.html5plus.org/#specification)，必须等plusready事件发生后才能正常使用，mui将该事件封装成了`mui.plusReady()`方法，涉及到HTML5+的api，建议都写在mui.plusReady方法中。

## 页面跳转

```js
mui.openWindow({
    url:new-page-url,//新页面地址
    id:new-page-id,//跳转页面id，注意一致
    //以下内容没用到的可省略
    styles:{
      top:newpage-top-position,//新页面顶部位置
      bottom:newage-bottom-position,//新页面底部位置
      width:newpage-width,//新页面宽度，默认为100%
      height:newpage-height,//新页面高度，默认为100%
      ......
    },
    extras:{//自定义扩展参数，可以用来处理页面间传值
      .....
    },
    createNew:false,//是否重复创建同样id的webview，默认为false:不重复创建，直接显示
    show:{
      autoShow:true,//页面loaded事件发生后自动显示，默认为true
      aniShow:animationType,//页面显示动画，默认为”slide-in-right“；
      duration:animationTime//页面动画持续时间，Android平台默认100毫秒，iOS平台默认200毫秒；
    },
    waiting:{
      autoShow:true,//自动显示等待框，默认为true
      title:'正在加载...',//等待对话框上显示的提示内容
      options:{
        width:waiting-dialog-widht,//等待框背景区域宽度，默认根据内容自动计算合适宽度
        height:waiting-dialog-height,//等待框背景区域高度，默认根据内容自动计算合适高度
        ......
      }
    }
})
```

## 页面嵌套

mui支持两种嵌套页面的方式：

### 1.HTML DOM innerHTML 属性

innerHTML在JS是双向功能：获取对象的内容 或 向对象插入内容；
如：

```html
<div id="a">这是内容</div>
<div id="b"></div>
```

我们可以通过 document.getElementById(‘a‘).innerHTML 来获取id为aa的对象的内嵌内容；
也可以对某对象插入内容，如 

```javascript
var div = ’这是被插入的内容’;
document.getElementById(‘b’).innerHTML=div;
```

这样就能向id为b的对象插入内容。



### 2.原生js操作dom方法：insertAdjacentHTML

insertAdjacentHTML() 将指定的文本解析为HTML或XML，并将结果节点插入到DOM树中的指定位置。它不会重新解析它正在使用的元素，因此它不会破坏元素内的现有元素。这避免了额外的序列化步骤，使其比直接innerHTML操作更快。

```javascript
mui('name')[0].insertAdjacentHTML('position', div);
```

mui('name')[0]为mui的绑定元素方法，name是元素的标签名称，id，class。[]内为下标，从0开始。

position是相对于元素的位置，并且必须是以下字符串之一：
beforebegin: 元素自身的前面。
afterbegin: 插入元素内部的第一个子节点之前。
beforeend: 插入元素内部的最后一个子节点之后。
afterend: 元素自身的后面。

div是定义的嵌套到页面的内容。

## mui.ajax()

```js
mui.ajax(url,{//url是请求的接口地址
    //以下为处理函数，部分可省略，也可添加其他
	data:{//随地址传入的参数
		username:'username',//注意逗号
		password:'password'
	},
	dataType:'json',//服务器返回json格式数据
	type:'post',//HTTP请求类型，post或get
	timeout:10000,//超时时间设置为10秒；          
	success:function(data){//接口请求成功后，返回响应的参数
        //注意data格式，若返回的是mgeids-boot系统中Result格式参数:
        data.status, //解析接口状态,true或false
        data.msg, //解析提示文本
        data.data.total, //解析数据长度
        data.data.list //解析数据列表，这里才是真正的json数据
		...
        //mgeds-boot响应成功后常用格式
        if(data.status){
            mui.each(data.data.list,function(i, item){
                ...//嵌套循环
            });
        }else{
            mui.alert(data.msg);
        }
	},
	error:function(){
		//异常处理；
		mui.alert('数据异常！');
	}
});
```

## 循环遍历

上面ajax响应成功后我们有个mui.each()方法，这就是mui的循环遍历方法。

示例：输出当前数组中每个元素的平方

```javascript
var array = [1,2,3]; //定义数组
mui.each(array,function(index,item){ 
    //function(index,item)为每个元素执行的回调函数；index表示当前元素的下标或key，item表示当前匹配元素
  console.log(item*item);
});
```

### 完整示例，mui.ajax() + mui.each() + innerHTML

```javascript
//详情页面插入资金信息				mui.ajax(ajaxUrl+'/rest/fund/fundReportController/getAssignDetailByAssign',{
	data:{fundAssignId: fundAssignId},
	dataType:'json',//服务器返回json格式数据
	timeout:10000,//超时时间设置为10秒；
	success:function(data){
		if (data.status) {
			var ftr = '<tr style="font-size: 20px;"><td>资金信息</td></tr>'
				 +'<tr><th style="width: 150px;">资金来源组织名称</th><th>资金属性</th>'
            	 +'<th style="width: 120px;">资金数额（万元）</th></tr>';
			mui.each(data.data.list, function(i, item){
				ftr +='<tr><td>'+item.fundOrganName+'</td><td>'
					+getfundType(item.fundProperty)+'</td><td>'
					+item.fundAmount+'</td></tr>';
			})
			fundBlockTable.innerHTML = ftr;
		}
	},
	error:function(){
		mui.alert("数据获取失败！","资金块信息");
	}
});
```

## 事件点击

当循环遍历输出列表信息时，常用onclick属性绑定按钮的点击事件。

```javascript
<button type="button" onclick="yes('+item.fundAssignId+')">确认</button>
<button type="button" onclick="no(\''+item.fundName+'\')">取消</button>
<script type="text/javascript">
    function yes(object){//object即为点击传入的参数
    	...
	}
    function no(object){//当参数为String型时，需注意要用引号传入，在HTML中注意使用转义字符"\"
    	...
	}
</script>
```


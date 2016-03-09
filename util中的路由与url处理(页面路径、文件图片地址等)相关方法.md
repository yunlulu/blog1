首先说明下 对util的访问：

    util.js：位于base-base-assets-js目录下
    util被挂载到了window对象的一个属性FS上，因此你可以通过window.FS.util来访问这两个提供的方法。

uitl提供了数据操作和请求的共用方法：比如Ajax请求、数据处理、信息提示框、模板处理、路由注册以及一些处理兼容的方法等等；下面将会介绍到一些常用的方法使用，其它的还请自己阅读文件。

1.util.tplRouterReg(tplNavSelector, mod, opts)

函数说明：
> tplRouterReg的实现用了460行代码，他是一个注册路由的底层函数，以实现各模块的切换。   
> 你可以在各模块（crm/manage...）中的app.js中发现他的使用身影。

使用说明：

    实现机制：
	util.tplRouterReg = (function(){
		//一些变量声明
		//1.tplRouterReg使用了backbone的路由模块，并将其挂载到了FS.tpl（下称tpl）的属性navRouter上；同时也将实例化了base-events模块挂在tpl.event上
		var NavRouter = Backbone.Router.extend( ... );
		var navRouter = new NavRouter();
            tpl.navRouter = navRouter;
            tpl.event = new Events(); 

		//2.接着声明了一些函数表达式
		var tplPreproccess，tplClear；
		var activeTpl = function(tplName,tplSelector, tplPath, mod){
			//3.这个函数完成的功能：
			设置document.title 隐藏页面的弹框和遮罩层 预览组件隐藏 相应模块对应的元素的显示与隐藏样式的添加与删除
		}
		var navTo = function(tplPath, navName, mod, flag){
			//4.这个函数完成的功能：
			body的class切换 遮罩层的初始化 未缓存tpl的移除清理或者隐藏缓存的tpl 模板处理（js css加载） 根据情况对activeTpl的调用
		}
		//5.最后返回匿名函数
		return function(tplNavSelector, mod, opts){
			//必须制定mod 否则直接退出
			//tplNavSelector 切换tpl的元素 若指定哈希，则创建a标签，设置href属性为tplNavSelector，然后循环该元素集合，注册路由进行一系列的url处理，最后调用FS.tplnavRouter.route(routerPath, navName, function (){ ... })
		}
		**opts参数唯一使用到的地方，就是用来确定tplPath和navName
			 if (opts.layout) {
                tplPath = opts.layout;
                navName = [mod, opts.layout.match(/^[^\/\.]+/)].join('-');
            }
	})()；

	使用实例，假设我需要再crm2模块下注册一个：
    util.tplRouterReg('#setting/contacts/field', 'crm2', {
        layout: 'main/main.html'，useLayoutClass: 'some className'
    });

2.util.regTplNav(elSelector,hlCls)

> 注册模板导航按钮，用于模板切换时高亮导航
> elSelector指定导航元素，hlCls高亮导航对应的className，默认“depw-menu-aon”

	实现机制：
	regTplNav:(function(){
		var store = [];
		//事件绑定
		tpl.event.on('switched', function (tplName, tplSelector) {
			//做的操作有：
				1.获取location中#以及后面的地址
				2.过滤出store中的项当中所指定的element被包含在body中，并赋值给store
				3.遍历store，如果指定项中的element属性对应的元素的href属性==1中获取的值，设置元素的hlCls样式，否则移除
				4.二级遍历store，设置元素的hlCls样式【设置二级导航样式】
		}
		return function(elSelector, hlCls){
			遍历每个elSelector，设置config,每个都包含element、hlCls、href、lightModel四个属性，并将config push到store中。
		}
	})();
	
	使用：
	这是base-app.js当中的一处使用，请参考：
	var tplNavEl = $('a.tpl-nav-l');
	util.regTplNav(tplNavEl, 'tnavon state-active');
	你可以这么理解，在点击模板切换后，判断点击的元素的href属性是否与location.href中‘#’以及后面的所截取的字符串是否相等，相等则添加指定的hlClas，否则移除。

3.getTplQueryParams(url) 
> 获取给定url或者页面href的query参数，确切来说主要匹配下面这种类型的url：   
> XXX#stream/followedfeeds/=/type-reply，处理后返回{type:reply} 若不是这种类型的url则返回null

4.getCurrentTplName()
> 获取当前页面的模板名
> 方法的实现机制是：先匹配去掉location.href后面’/=/‘即后面的部分，最后截取’#‘后面的部分，若剩下的hash中含有’/‘，则将其转换为’-‘;   

	假设你所访问的页面你是：http://www.fxiaoke.com/XV/Home/Index#stream
	方法将返回'stream'   
	如果地址是这种的：http://www.fxiaoke.com/XV/Home/Index#stream/followedfeeds/=/type-reply
	方法将返回stream-followedfeeds

##### 附件类地址获取相关方法

5.getFscTempFileUrl(filePath, name, isAttachment)
> 获取filePath的路径   
> filePath包含文件扩展名  比如 test.js  请不要包含’./‘这类  
> name文件名 不带扩展名
> isAttachment表示是否为附件，控制url是否附带isAttachment=1参数

	FS.util.getFscTempFileUrl('asci.doc','总结',true)
	// "http://www.fxiaoke.com/FSC/EM/File/DownloadTempFile?TempFileName=asci&Name=%E6%80%BB%E7%BB%93.doc&isAttachment=1&traceId=S-E.fs.3925-39426377"
	
同样的`getFscTempFileViewUrl(filePath)`用来获取文件的预览路径,`getFscFileUrl(filePath, name, isAttachment)`的使用与getFscTempFileUrl相同，只是返回的url带有不同的参数。其他获取地址类型的方法还有：

	getDfLink(id,name,isAttachment,extendInfo)
	getFscLink(id, name, isAttachment)
	getFscLinkByToken(id, type)
	getDfGlobalLink(id,name,isAttachment,extendInfo)
	getAvatarLink(path,sizeType)
	getExternalLink(path)
	getExternalLink(path)
	getBaichuanCrossImageUrl(path, type)
	getBaichuanCrossDownloadUrl(path, name, type)
	getBaichuanCrossMp3Url(path, type)
	getBaichuanCrossAvatarLink(path, sizeType)
	
6.getFileExtText(fileName) | getFileNamePath(fileName)

> 前者获取文件扩展名，而后者获取文件名(不包含扩展)   
> 比如`getFileExtText('abd.js')`  => 'js'     
> 而`getFileNamePath('abd.js')`  => 'abd'     
> 这两个都是通过lastIndexOf('.')确定索引，然后选择截取前部分还是后部分

7.getUploadType(fileName)
> 获取上传文件的类型   
> 如果为jpg | jpeg | png | bmp | gif 则返回img类型，其他则返回‘attach’类型   

8.queryToJson(query) 和 jsonToQuery(c,e)
> 前者把a=b&c=2&name=dd转换成 {a:b,c:2,name:dd}这种形式   
> 后者的功能则与前者相反，它将json对象转换为用&符号连接的url参数   
> 另外后者需要注意的一个点就是参数c是json对象，参数e从源代码的实现来看，没有起到实质性作用，传与不传都不影响结果。

---End---

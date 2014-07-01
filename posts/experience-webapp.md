#移动开发个人总结
##过去的经验陷井与思维定势

在开发官网的时候，引入iscroll5, 由此引发了一些问题。
* 光标跳动/文本框消失
* 滑动隐藏虚拟键盘
* 引发的性能问题
google之，发现这些问题很普遍。解决方案也很多，总之是个相应麻烦与
头疼的问题。

后来想，当时我为什么要用iscroll呢，仅仅是因为过去使用过，觉得很好
用，体验也不错。用在这里合适吗？


直觉用原生滚动会更好，[ overflow:scrolling 兼容性检测报告](http://www.quirksmode.org/css/css2/mobile.html)
**ios 有个小技巧**
```css
	overflow-y: scroll; /* has to be scroll, not auto */
	overflow-x: hidden;
	-webkit-overflow-scrolling: touch; // for safari scroll bug.
```
![alt text][https://raw.githubusercontent.com/heyi/blog/master/imgs/mobile_overflow.png,"overflow scrolling detected"]
如图，不支持overflow scrolling ,overflow auto 的手机或者浏览器主要集中android 3 以下的版本。 现在android 基本是4以上，2年前的旧手机普遍是3左右;
值得注意的是uc ,在国内市场占有率是比较高的。这也是一个坑，所以在决定方案的时候，可以考虑进去。 
得出的结论是，绝大多数情况下，用原生的滚动会更合适。

##用sass组织css非常方便
```
.concat-us {
	.logo {
		background-image: url(../images/concat-us.png);
		height: 180px;
	}
	.info {
		padding: 12px;
		h3 { margin-bottom: 12px; }
		i.iconfont {
			font-size: 1.2rem;
			color : #4fbcbe;
		}
		a.email-text {
			color : #4fbcbe;
		}
	}
}
```
项目中的部分sass代码片断，这种组织的好处就是方便管理，互不影响。
在设计风格多变的页面中有很大管理维护优势，缺点是提取共用性的css会比较困难.

##使用渐进式 JPEG 能提升用户体验
[关于progressive-jpeg](http://www.webmonkey.com/2013/01/the-return-of-the-progressive-jpeg/)
[简单说明](http://www.biaodianfu.com/progressive-jpeg.html)
我第一个想法就是，这个渐进式 jpeg确实很不错，有法子在部署的时候自动转换吗？
我仔细看了下部署脚本，其中有用到[grunt-contrib-imagemin](https://github.com/gruntjs/grunt-contrib-imagemin#progressive-jpg)
看官方文档是有这个选项的，而且是默认选中的，很贴心了。

##虚拟键盘遮挡输入框问题。
这个问题也是很普遍的， 具体表现为如果容器的布局是 --absolute-- 或者 --fixed--,
点击输入框时，弹出的虚拟键盘会遮挡住输入框。我这里的表现是虚拟键盘会顶高页面,但页面会反弹回来(测试手机是红米)。

网上的解决方案不外乎二种,一种是使用iscroll ,在窗口resize事件中，重置容器的高度，
刷新iscroll,同时滚动到最底层。
另一种就是不使用iscroll ,使用定时器监控dom变化，改变容器的布局。
这二种方案的缺点是很明显的，带来的弊端且直无法忍受。

想了想，最终的法子如下：
编写了个简单的指令
```javascripts
app.directive('scrollHere', function(fmyApi) {
  return {
    restrict: 'A',
    link: function($scope, element, attrs) {
			element.on('click', function(e) {
				setTimeout(function() {
					element[0].scrollIntoView(true);
				},500);
			});
			$scope.$on('$destroy', function() {
				element.off("click");
			});
		}
	};
});
```
**应用场景：**
```html
<div class="input-container sbtn" scroll-here>
			<input type="text" name="mobile" class='form-control' ng-model='user.mobile' required placeholder='输入手机号' ng-pattern='/^1\d{10}$/'>
			<button ng-click='getMobileCode()' ng-disabled="form4.mobile.$invalid || sending">
				{{sending ? '正发送中..' : '发送验证码'}}
			</button>
			<span class="tip" ng-show='sending && countDown != 0'>{{countDown}}秒后可以重新获取验证码</span>
			<span class="error" ng-show='form4.mobile.$error.pattern'>不是手机号码!</span>
		</div>
```

可以看出，不是很严谨，因为无法捕获虚拟键盘弹出事件。


---
layout: post
title:  扇形绘制
date:   2014-03-30 10:00:00
categories: study
---

扇形的绘制，常常在游戏的技能冷却的时候，或者是倒计时的时候，特效上会需要用到。在网上搜索了一下，发现了一个比较好用的扇形封装类，特此记下来，以备后用。

先说说原理吧，主要是用到Graphice的moveTo()，lineTo()方法。不停的从中心点绘制一条定长的直线，这一系列直线组合在一起就是扇形了。

封装好的两个类，首先是CoolDownEffect_fan
{% highlight actionscript %}
package 
{
	
	import flash.display.BitmapData;
	import flash.display.DisplayObject;
	import flash.display.DisplayObjectContainer;
	import flash.display.Shape;
	import flash.display.Sprite;
	import flash.events.Event;
	import flash.filters.ColorMatrixFilter;
	import flash.text.TextField;
	import flash.text.TextFormat;
	import flash.utils.getTimer;
	
	/**
	 * 冷却特效-扇形
	 */
	public class CoolDownEffect_fan
	{
		/**
		 * 目标
		 */
		private var _target:DisplayObject;
		
		/**
		 * 将目标对象的一个灰色背景显示到底部.
		 */
		private var targetBg:Sprite;
		
		/**
		 * 遮罩(_target被遮罩),在此遮罩上绘制不断改变的扇形,以达到特效效果.
		 */
		private var fanMask:Shape;
		
		/**
		 * 冷却时间显示
		 */
		private var coolText:TextField;
		
		/**
		 * 开始时间
		 */
		private var startTime:int;
		
		/**
		 * 持续时间
		 */
		private var _duration:int;
		
		/**
		 * 初始角度
		 */
		private var _initAngle:Number;
		
		/**
		 * 冷却特效显示完成时的回调.
		 */
		private var _completeCallback:Function;
		
		/**
		 * 冷却倒计时显示
		 */
		private var _coolTimeTextFormat:TextFormat;
		
		/**
		 * 是否正在播放特效
		 */
		private var _isPlaying:Boolean;
		
		/**
		 * 扇形的半径
		 */
		private var radius:Number;
		
		/**
		 * 扇形的中心点X坐标
		 */
		private var centerX:Number;
		
		/**
		 * 扇形的中心点Y坐标
		 */
		private var centerY:Number;
		
		/**
		 * 上一次绘制扇形的角度
		 */
		private var lastAngle:Number;
		
		/**
		 * 灰色滤镜
		 */
		public static const GRAY_FILTERS:Array=[
			new ColorMatrixFilter([
				0.25,         0.25,         0.25,         0,         0,
				0.25,         0.25,         0.25,         0,         0,
				0.25,         0.25,         0.25,         0,         0,                 
				0,            0,            0,            1,         0
			])
		];
		
		/**
		 * 将倒计时文本显示中心点
		 * 
		 */
		private function toCenterCoolText():void{
			coolText.width = coolText.textWidth + 4;
			coolText.height = coolText.textHeight + 4;
			coolText.x = targetBg.x + (targetBg.width - coolText.width) / 2;
			coolText.y = targetBg.y + (targetBg.height - coolText.height) / 2;
		}
		
		/**
		 * 
		 * @param event
		 * 
		 */
		private function enterFrameHandler(event:Event):void{
			var interval:int= getTimer() - startTime;
			if(interval >= _duration){
				stop();
			}
			
			var endAngle:Number = _initAngle + 360 * (interval / _duration);
			ShapeDraw.drawFan(fanMask.graphics, radius, lastAngle, endAngle, centerX, centerY, 4);
			
			if(_coolTimeTextFormat){
				var remainingTime:int = Math.ceil((_duration - interval) / 1000);
				if(remainingTime != int(coolText.text)){
					coolText.text = String(remainingTime);
					toCenterCoolText();
				}
			}
			lastAngle = endAngle;
		}
		
		/**
		 * 播放扇形冷却特效
		 * @param target	目标
		 * @param duration	持续时间
		 * @param initAngle	初始角度
		 * @param completeCallback	冷却特效显示完成时的回调.
		 * @param coolTextTextFormat	冷却倒计时显示
		 * @param coolTextFilters
		 * @param fanFilters
		 * 
		 */
		public function play(target:DisplayObject, duration:int, initAngle:Number = 0, completeCallback:Function = null, 
				coolTextTextFormat:TextFormat = null, coolTextFilters:Array = null, fanFilters:Array = null):void{
			var parent:DisplayObjectContainer = null;
			
			if(duration <= 0){
				return;
			}
			
			if(target && target.stage && !_isPlaying){
				_isPlaying = true;
				_target = target;
				_duration = duration;
				_initAngle = initAngle;
				_completeCallback = completeCallback;
				_coolTimeTextFormat = coolTextTextFormat;
				parent = target.parent;
				
				var targetBmp:BitmapData = new BitmapData(target.width, target.height, true, 0);
				targetBmp.draw(target);
				//BitmapDraw.drawBitmap(targetBg.graphics, targetBmp);
				
				targetBg.graphics.beginBitmapFill(targetBmp);
				targetBg.graphics.drawRect(0,0,target.width, target.height);
				
				targetBg.filters = fanFilters || GRAY_FILTERS;
				targetBg.x = target.x;
				targetBg.y = target.y;
				parent.addChildAt(targetBg, parent.getChildIndex(target));
				if(target is Sprite){
					(target as Sprite).hitArea = targetBg;
				}
				
				fanMask.x = targetBg.x;
				fanMask.y = targetBg.y;
				parent.addChild(fanMask);
				target.mask = fanMask;
				
				if(coolTextTextFormat){
					coolText.filters = coolTextFilters;
					coolText.defaultTextFormat = coolTextTextFormat;
					coolText.setTextFormat(coolTextTextFormat);
					coolText.text = String(duration / 1000);
					toCenterCoolText();
					parent.addChildAt(coolText, (parent.getChildIndex(target) + 1));
				}
				
				radius = Math.sqrt(target.width * target.width + target.height * target.height) / 2;
				centerX = target.width / 2;
				centerY = target.height / 2;
				startTime = getTimer();
				lastAngle = initAngle;
				fanMask.graphics.clear();
				fanMask.graphics.beginFill(16711680, 0.5);
				targetBg.addEventListener(Event.ENTER_FRAME, enterFrameHandler);
			}
		}
		
		/**
		 * 停止特效播放
		 * 
		 */
		public function stop():void{
			if(_isPlaying){
				_isPlaying = false;
				targetBg.removeEventListener(Event.ENTER_FRAME, enterFrameHandler);
				if(_target is Sprite){
					(_target as Sprite).hitArea = null;
				}
				_target.mask = null;
				
				targetBg.graphics.clear();
				if(targetBg.parent){
					targetBg.parent.removeChild(targetBg);
				}
				
				fanMask.graphics.clear();
				if(fanMask.parent){
					fanMask.parent.removeChild(fanMask);
				}
				
				if(coolText.parent){
					coolText.parent.removeChild(coolText);
					coolText.filters = null;
				}
				
				if(_completeCallback != null){
					_completeCallback();
				}
			}
		}
		
		/**
		 * 冷却特效-扇形
		 * 
		 */
		public function CoolDownEffect_fan(){
			targetBg = new Sprite();
			targetBg.mouseEnabled = false;
			fanMask = new Shape();
			coolText = new TextField();
			coolText.mouseEnabled = false;
		}
		
		/**
		 * 目标
		 * @return 
		 * 
		 */
		public function get target():DisplayObject{
			return _target;
		}
	}
}
{% endhighlight %}

然后是ShapeDraw，ShapeDraw会在每帧被CoolDownEffect_fan调用。
{% highlight actionscript %}
package 
{
	import flash.display.Graphics;
	import flash.geom.Point;
	
	/**
	 * 形状绘制
	 */
	public class ShapeDraw
	{
		/**
		 * 获取圆上的一点
		 * @param radius 半径
		 * @param radian 弧度
		 * @param centerX 中心点X坐标
		 * @param centerY 中心点Y坐标
		 * @param point 运算完成后会将结果值设置到此对象上
		 * @return 圆上的一点
		 * 
		 */
		public static function getPointOfCircleByRadian(radius:Number, radian:Number, 
					centerX:Number, centerY:Number, point:Point):Point{
			if(!point){
				point=new Point();
			}
			
			point.x = radius * Math.cos(radian);
			point.y = radius * Math.sin(radian);
			point.offset(centerX, centerY);
			
			return point;
		}
		
		/**
		 * 获取圆上的一点
		 * @param radius 半径
		 * @param angle 角度
		 * @param centerX 中心点X坐标
		 * @param centerY 中心点Y坐标
		 * @param point 运算完成后会将结果值设置到此对象上
		 * @return 圆上的一点
		 * 
		 */
		public static function getPointOfCircleByAngle(radius:Number, angle:Number, centerX:Number, centerY:Number, point:Point):Point{
			return getPointOfCircleByRadian(radius,angle / 180 * Math.PI,centerX,centerY,point);
		}
		
		public static function drawFan(graphics:Graphics, radius:Number, startAngle:Number, endAngle:Number,
					centerX:Number = 0, centerY:Number = 0, step:Number = 5):void{
			if(startAngle == endAngle || step == 0){
				return;
			}
			
			const PI:Number=Math.PI;
			const PI2:Number=PI*2;
			
			var startR:Number=startAngle / 180 * PI;
			var endR:Number=endAngle / 180 * PI;
			step=step / 180 * PI;
			
			var p:Point=new Point();
			var radian:Number=0;
			
			graphics.moveTo(centerX, centerY);
			if(startR < endR){
				if(endR - startR >= PI2){
					startR=0;
					endR=PI2;
				}
				if(step < 0){
					step=-step;
				}
				
				radian=startR;
				do{
					getPointOfCircleByRadian(radius,radian,centerX,centerY,p);
					graphics.lineTo(p.x,p.y);
					
					radian += step;
				}while(radian < endR);
			}else{
				if(startR - endR >= PI2){
					startR=PI2;
					endR=0;
				}
				if(step > 0){
					step=-step;
				}
				
				radian=startR;
				do{
					getPointOfCircleByRadian(radius,radian,centerX,centerY,p);
					graphics.lineTo(p.x,p.y);
					
					radian += step;
				}while(radian > endR);
			}
			
			getPointOfCircleByRadian(radius,endR,centerX,centerY,p);
			graphics.lineTo(p.x, p.y);
			graphics.lineTo(centerX, centerY);
		}
	}
}
{% endhighlight %}

下面的外部使用的类，或者叫做测试类吧
{% highlight actionscript %}
package
{
	import flash.display.DisplayObject;
	import flash.display.Shape;
	import flash.display.Sprite;
	import flash.display.StageScaleMode;
	import flash.events.Event;
	import flash.text.TextFormat;
	import flash.utils.setTimeout;
	
	public class TestOne extends Sprite
	{
		
		[ Embed(source="skill.swf", symbol="Skill_600001") ] 
		private var SKILL:Class;
		
		public function TestOne()
		{
			stage.scaleMode=StageScaleMode.NO_SCALE;
			
//			测试扇形绘制，请把下面两行的注释打开
//			setTimeout(init, 5000);
//			return;
			
//			具体情况下使用扇形倒计时
			var sp:DisplayObject = new SKILL as DisplayObject;
			sp.x=sp.y=100;
			addChild(sp);
			var fanEft:CoolDownEffect_fan=new CoolDownEffect_fan();
			//参数:目标对象,持续时间(毫秒),初始角度,播放完成时的回调,倒计时的文本格式,倒计时文本的滤镜,扇形背景的滤镜
			fanEft.play(sp,3000,-90,playComplete,new TextFormat(null,18,0x00ffff,true));
			function playComplete():void{
				fanEft.play(sp,3000,-90,playComplete,new TextFormat(null,18,0xff00ff,true));
//				trace("complete");
			}
		}
		
		/**
		 * 测试
		 */		
		private function init():void
		{
			var sb:Shape = new Shape();
			sb.graphics.lineStyle(3);
			sb.graphics.moveTo(200,200);
			sb.graphics.lineTo(200,100);
			addChild(sb);
			sb.addEventListener(Event.ENTER_FRAME, onhandle);
			var xx:Number;
			var yy:Number;
			var endAngle:int = 0;
			var starAngle:int = 90;
			function onhandle(e:Event):void
			{
				if(starAngle < endAngle)
				{
					sb.removeEventListener(Event.ENTER_FRAME, onhandle);
				}
				else
				{
					starAngle -= 1;
					xx = 200+100*Math.cos(starAngle/180*Math.PI);
					yy = 200-100*Math.sin(starAngle/180*Math.PI);
					sb.graphics.lineTo(xx, yy);
					sb.graphics.moveTo(200,200);
				}
			}
		}
		
	}
}
{% endhighlight %}


参考资料
<a href="http://bbs.9ria.com/forum.php?mod=viewthread&tid=220843&highlight=扇形" target="_blank">http://bbs.9ria.com/forum.php?mod=viewthread&tid=220843&highlight=扇形</a>

源代码下载
<a href="../../../../download/sector-src.rar" target="_blank">源代码</a>
---
layout: post
title:  富文本实现框架--RichTextField
date:   2014-12-07 16:37:00
categories: study
---

说起RichTextField，在09年的时候就接触了，使用过一次，感觉当时没有遇到这么多的问题啊，最近开发聊天系统。项目组没有人，只有自己亲自出马了。在选择技术方案的时候，纠结是使用TLF，还是使用第三方开源库呢。结合项目情况，比较后发现，采用RichTextField吧，TLF主要是要再倒入一个库，大约有200K，太大了。随后就采用了RichTextField，隔的时间有些久了，做的过程中遇到这些坑，mark下来。

RichTextField的实现思路，是分两层，一层显示文本，一层显示表情，插入表情的时候，我们可以很清楚的知道插入的位置，然后记下当前位置和表情，然后在文本的这个地方空出足够的宽度，显示表情。

1，关于RichTextField2.0在flex sdk4.6下表情显示错乱的问题

解决方法：

SpriteRenderer.as类

{% highlight actionscript %}
private function renderSprite(sprite:DisplayObject, index:int):void
        {            
            var rect:Rectangle = textRenderer.getCharBoundaries(index);    
            if (rect != null)
            {
                sprite.x = (rect.x + (rect.width - sprite.width) * 0.5 + 0.5) >> 0;
                var y:Number = (rect.height - sprite.height) * 0.5;
                var lineMetrics:TextLineMetrics = textRenderer.getLineMetrics(textRenderer.getLineIndexOfChar(index));
                //make sure the sprite's y is not smaller than the ascent of line metrics
                if (y + sprite.height < lineMetrics.ascent) y = lineMetrics.ascent - sprite.height;
                sprite.y = (rect.y + y + 0.5) >> 0;
                //flex sdk 4.6添加,否则有显示bug
                sprite.y += -_spriteContainer.y;
                _spriteContainer.addChild(sprite);
            }
        }
{% endhighlight %}
参考blog	
http://blog.sina.com.cn/s/blog_4b6331fd0101goz3.html

2，针对在输入框中，输入文本和插入表情的时候，光标会突然变大的解决方法是

在RichTextField.as类中的，calcPlaceholderFormat方法
{% highlight actionscript %}
private function calcPlaceholderFormat(width:Number, height:Number):TextFormat
  {
   var format:TextFormat = new TextFormat();
   format.color = _placeholderColor;
//	 format.size = height + 2 * _placeholderMarginV;
   if(type == DYNAMIC)
    format.size = height + 2 * _placeholderMarginV;
   
   //calculate placeholder text metrics with certain size to get actual letterSpacing
   _formatCalculator.setTextFormat(format);
   var metrics:TextLineMetrics = _formatCalculator.getLineMetrics(0);
   
   //letterSpacing is the key value for placeholder
   format.letterSpacing = width - metrics.width + 2 * _placeholderMarginH;
   format.underline = format.italic = format.bold = false;
   return format;
  }
{% endhighlight %}

3，针对在输入框中，输入文本和插入表情，字数超过显示区域，输入框默认会自动向左把文本内容移动左边一些，而这个时候表情显示就会位置不对。
解决方法是：

在SpriteRenderer.as类中，renderSprite方法，修改为
{% highlight actionscript %}
  private function renderSprite(sprite:DisplayObject, index:int):void
  {
   var rect:Rectangle = textRenderer.getCharBoundaries(index);
   if (rect != null)
   {
    var offx:int = textRenderer.scrollH;
    var spritex:int = ((rect.x + (rect.width - sprite.width) * 0.5 + 0.5) >> 0) - offx;
    var y:Number = (rect.height - sprite.height) * 0.5;
    var lineMetrics:TextLineMetrics = textRenderer.getLineMetrics(textRenderer.getLineIndexOfChar(index));
    //make sure the sprite's y is not smaller than the ascent of line metrics
    if (y + sprite.height < lineMetrics.ascent) y = lineMetrics.ascent - sprite.height;
    sprite.y = (rect.y + y + 0.5) >> 0;
    sprite.y += -_spriteContainer.y;
    sprite.x = spritex;
    if(spritex>=0)
     _spriteContainer.addChild(sprite);
   }
  }
{% endhighlight %}


源代码下载
<a href="../../../../download/RichTextField.rar" target="_blank">源代码</a>

项目运行注意事项，Test.as是文档类，打开fla文件，然后发布，就可以看到效果，代码查看请使用EditPlus,或者在Flashbuilder里，建立一个工程，导入，查看代码
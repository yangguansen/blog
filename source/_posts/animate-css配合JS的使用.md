---
title: animate.css配合JS的使用
date: 2016-09-03 14:31:49
tags:
	- JavaScript
---
刚刚在前端群里看到有人问animate如何配合JS使用，再给他讲解了一下后觉得应该在博客中记录下来，以能帮到更多人的更好的使用animate.css，其实这个CSS库在动效方面蛮好用的，记得我之前面试公司的考试就是做一个动效类的H5，那时候我用的是jqueryUI的库，放在手机上非常的庞大，也略卡顿，划过下一页之后再滑到上一页，就没有动效了。后来去了公司之后才知道的animate.css类库，于是自己又结合JS封装一下方法，页面之间的滑动不管滑多少次都会重新展示动画，废话不多说，上代码：
1、首先引入animate.CSS库和zepto库。

2、HTML的代码这样写：
```html
<div class="page page1 animated fadeIn hide">						
	<img src="p11.png" class="pos_abs animated p11" data-class="bounceInDown" data-delay="0.3s">
    <img src="p12.png" class="pos_abs animated p12" data-class="bounceIn" data-delay="1.3s">
    <img src="p13.png" class="pos_abs animated p13" data-class="fadeInUp" data-delay="2.3s">
	<img src="p14.png" class="pos_abs animated p14" data-class="fadeInUp1 infinite" data-delay="3.3s" alt="">
</div>
```
animated是需要有动效的元素，data-class是动画效果名称，data-delay是元素延时时间，通过延时时间的排列来达到展示动画的衔接。

3、
```javascript
(function(globle){
    var ff = (function(){
        return{
            num: 1,
            divHtmlArr:[],
            total : $(".page").length,
            next : function(){
                if(this.num < this.total){
                    $(".page" + this.num).addClass("fadeOut hide");
                    this.initHtml(this.num-1, $(".page" + this.num));
                    this.num++;
                    this.setAnimation(this.num);
                }
            },
            setHtml : function (index,div){
                if(!this.divHtmlArr[index]){
                    this.divHtmlArr[index] = div; //将每一屏代码存放到数组
                }   
            },
            getHtml : function (div){   //获取html代码
                return $(div).html();
            },
            initHtml : function (index,div){    //初始化html代码
                $(div).html(this.divHtmlArr[index]);
            },
            last : function(){
                if(this.num > 1){
                    $(".page" + this.num).addClass("fadeOut hide");
                    this.initHtml(this.num-1, $(".page" + this.num));
                    this.num--;
                    this.setAnimation(this.num);
                }
            },          
            setAnimation : function(pageNum){
                var html = this.getHtml($(".page" + pageNum));
                this.setHtml(pageNum-1, html);
                $(".page" + pageNum).removeClass("fadeOut hide");
                var AnimateImg = $(".page" + pageNum).find(".animated");
                $(AnimateImg).each(function(index, value){
                    var animationStyle = $(this).data("class");
                    var animationDelay = $(this).data("delay");
                    $(this).addClass(animationStyle).css({
                        "animation-delay":animationDelay,
                        "-webkit-animation-delay":animationDelay
                    });
                })  
            },
            IsPC : function() {
                var userAgentInfo = navigator.userAgent;
                var Agents = ["Android", "iPhone",
                            "SymbianOS", "Windows Phone",
                            "iPad", "iPod"];
                var flag = true;
                for (var v = 0; v < Agents.length; v++) {
                    if (userAgentInfo.indexOf(Agents[v]) > 0) {
                        flag = false;
                        break;
                    }
                }
                return flag;
            },
            autoView : function(userAgent){
                var screen_w=parseInt(window.screen.width),scale=screen_w/640;
                if(/Android (\d+\.\d+)/.test(userAgent)){
                    var version=parseFloat(RegExp.$1);
                    $("head").prepend(version>2.3?'<meta id="auto_view" name="viewport" content="width=640, initial-scale = '+scale+',user-scalable=1, minimum-scale = '+scale+', maximum-scale = '+scale+', target-densitydpi=device-dpi">':'<meta name="viewport" content="width=640, target-densitydpi=device-dpi">');
                }else{
                    $("head").prepend('<meta id="auto_view" name="viewport" content="width=640, initial-scale = '+scale+' ,minimum-scale = '+scale+', maximum-scale = '+scale+', user-scalable=no, target-densitydpi=device-dpi">');
                };
                if(this.IsPC()){
                    document.body.style.margin = "0px auto";
                }
            },
            loadImg : function(){
                var totalNum = 0;
                var jiazai = 0;
                var imgs = document.getElementsByTagName("img");
                var imgLen = imgs.length;        
                function setBaiNum(bai){
                    $("#baifen_txt").text(bai);
                    if(bai == 100){
                        setTimeout(function(){
                            $(".p1_close").addClass('hide');                                
                            ff.setAnimation(1);
                            
                        }, 500);
                        
                    }
                }
                for(var i = 0; i < imgLen; i++){
                    if(imgs[i].complete){
                        jiazai++;
                        var baifenbi = Math.floor(jiazai/imgLen*100);
                        setBaiNum(baifenbi);
                        continue;
                    }
                    imgs[i].onload = function(){
                        jiazai++;
                        var baifenbi = Math.floor(jiazai/imgLen*100);
                        setBaiNum(baifenbi);
                        
                    }
                }
            },
            toPage: function(num){
                $(".page" + this.num).addClass('hide');
                ff.setAnimation(num);
                ff.num = num;
            }
        }
    })();
    globle.ff = ff; 
})(this);
```
这是我对整个动效js的封装的一个对象，在编写好HTML后，直接复制这个代码，当前页滑到下一页可以调用ff.next（），若要通过按钮的点击来滑到某一页就调用ff.toPage(num)。setHtml()、getHtml()的方法是为了初始化HTML代码，以便滑到上一页的时候可以再次展示动效。
注意一点：绑定点击事件用$(document).on("click"，“.class”，function(){})的方式来绑定。如果用普通的$(".class").click(function(){})这种方法来绑定的话，setHtml()方法会把绑定在元素上的事件也抹掉。
(完)
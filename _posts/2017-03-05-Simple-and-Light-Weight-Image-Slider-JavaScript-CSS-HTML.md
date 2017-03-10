---
layout: post
title: Simple and Lightweight JavaScript Image Slider by Ravinder Payal
author: ravinder_payal
---
<p>
Hi Guys, today, we are going to develop and a simple image-slider with the help of CSS and JavaScript (pure JavaScript). SOme of us will be thinking that why re-invent the wheel when libraries like JSSOR, and JQUERY carousel plugins are already available:-
</p>
1.	No worries about licensing.
2.	To get a clear understanding of how slide shows and transitions work on web pages.
3.	Light weight solutions, because the ready to use libraries have a lot of boiler plate code.
4.	To have complete control over code.
5.	Tell me if you know more.

>Wait!! If you are using Angular 2, we have another article totally focused on Slider for Angular 2. First of all we will prepare our HTML structure.

Slider.html
```html
<div id="slider">
<div class="sliderArrows">
                <a onclick="backWard()"><</a>
                <a onclick="forWard()">></a>
</div>
<ul class="slideShow">
<li class="active"><img src="images/1.jpg" /></li>
<li><img src="images/2.jpg" /></li>
<li><img src="images/3.jpg" /></li>
</ul>
</div>
```
Now, as we can see our HTML code is self explanatory. We can add any number of images in Unordered List(UL.slideShow) with IMG tag. The divider tag(DIV.sliderArrows) contains the forward and backward arrows. Right now we have used greater than and lower than signs, but any icon or sign can be used. Now come to the CSS part. First look at the whole CSS file before break it apart.

124 lines of CSS code
```css
/*---------------------------Image Slider--------------------------*/
#slider{
    height:400px;
    width:100%;
    font-size: 500%;
    color:#000;
    z-index: 0;
    background: #fff;
    position: relative;
}

#slider ul{
    display: flex;
    padding: 0;
    justify-content: center;
    margin: 0;
}
#slider ul li {
    list-style: none;
    width: 100%;
    display: none;
    /*transition: cubic-bezier(0.6, -0.61, 0, 1.34) all 0.7s;*/
}
#slider ul li img{
    max-width:100%;
    min-width:100%;
    height: 100%;
    display: block;
}

#slider ul li.active{
    display: inline-block;
}
#slider ul li.backward{
    animation: slideShow0 1.2s cubic-bezier(0.6, -0.61, 0, 1.34);
}
#slider ul li.forward{
    animation: slideShow1 1.2s cubic-bezier(0.6, -0.61, 0, 1.34);
}
#slider ul li.animateBack{
    display: inline-block;
    position: absolute;
    animation: slideShow2 1s cubic-bezier(0.38,-0.74, 0, 1.29) forwards;
    animation-delay: 0.1s;
}
#slider ul li.animateForward{
    display: inline-block;
    position: absolute;
    animation: slideShow3 1s cubic-bezier(0.38,-0.74, 0, 1.29)  forwards;
    animation-delay: 0.1s;
}
#slider .sliderArrows{
    display:flex;
    position: absolute;
    flex-flow:row wrap;
    justify-content:space-between;
    width: 100%;
    align-items:center;
    height: 100%;
    z-index: 101;
}
#slider .sliderArrows a{
    cursor:pointer;
    font-weight: 900;
}


@keyframes slideShow0{
    0%{
         translate:-1000px;
         visibility: hidden;
         opacity: 0;
    }
    100%{
        translate:0px;
        visibility: visible;
        opacity:1;
    }

}


@keyframes slideShow1{
    0%{
         translate:1000px;
         visibility: hidden;
         opacity: 0;
    }
    100%{
        translate:0px;
        visibility: visible;
        opacity:1;
    }

}

@keyframes slideShow2{
     0%{
        display: inline-block;        
        translate:0px;
        visibility: visible;
        opacity: 1;
     }
    100%{
        translate:-1000px;
        visibility: hidden;
        opacity: 0;
    display: none;
        /*display: none;*/
     }

}

@keyframes slideShow3{
     0%{
        display: inline-block;        
        translate:0px;
        visibility: visible;
        opacity: 1;
     }
    100%{
        translate:1000px;
        visibility: hidden;
        opacity: 0;
    display: none;
        /*display: none;*/
     }
}
/*---------------------------Image Slider Ends----------------------------*/
```
One thing important in this code is that we have used flex-box property for arrow container, which makes our arrows responsive and align vertically and horizontally at exactly right places. Second thing is that we have defined animation key frames for 4 types of transitions we will be adding to our Slider. Note: - We can add any number of different transitions, but showing different types of transition effects is not in the aim of this tutorial, and we will be doing to a separate tutorial for that. Let’s define our key frames.

First key frame is `SlideShow0`:-
```css
@keyframes slideShow1{
    0%{
         translate:1000px;
         visibility: hidden;
         opacity: 0;
    }
    100%{
        translate:0px;
        visibility: visible;
        opacity:1;
    }
}
```
This frame is used for animating the appearing image/element (In case of non image slide-shows). It first translates/moves the image to 1000px right side from its current position, and then it brings back to its current position which gives image sliding effect (you name it better). Along with position transition, it also gives an opacity transition. Opacity and Visibility can be used interchangeably.
```css
@keyframes slideShow1{
    0%{
         translate:1000px;
         visibility: hidden;
         opacity: 0;
    }
    100%{
        translate:0px;
        visibility: visible;
        opacity:1;
    }

}
```
SlideShow1 is for backward animation of appearing image. Similarly, SlideShow2 and 3 are for backward and forward animation of disappearing images.
```css
#slider ul li.active{
    display: inline-block;
}
#slider ul li.backward{
    animation: slideShow0 1.2s cubic-bezier(0.6, -0.61, 0, 1.34);
}
#slider ul li.forward{
    animation: slideShow1 1.2s cubic-bezier(0.6, -0.61, 0, 1.34);
}
#slider ul li.animateBack{
    display: inline-block;
    position: absolute;
    animation: slideShow2 1s cubic-bezier(0.38,-0.74, 0, 1.29) forwards;
    animation-delay: 0.1s;
}
#slider ul li.animateForward{
    display: inline-block;
    position: absolute;
    animation: slideShow3 1s cubic-bezier(0.38,-0.74, 0, 1.29)  forwards;
    animation-delay: 0.1s;
}
```
It’s the use of key frames. One thing worth noting is that we are using forwards property of animation-fill-mode ( animation-fill-mode: forwards;) and cubic-bezier is used for defining the curve which the timing function of animation will follow. Note: - Chrome has a good graphical tool for defining curve, and shows a live demo of animation with defined curve. Just click on the curve sign appearing before curve function in CSS tab in developer tools.
Let’s move on to JAVASCRIPT code.

JavaScript
```javascript
class Slider{
    constructor(){
            this.slideShow=document.getElementById("slideShow");
            this.currentElement=0;
            this.lis=this.slideShow.childNodes;
            this.number=this.lis.length;
            this.autoMode=false;
    }
     backWard(){
         if(this.auto)clearInterval(this.interval);
        this.currentElement=this.currentElement-1;
        if(this.currentElement<0){
            this.currentElement=this.number-1;
        }
        this.removeClasses();
        var prev=this.currentElement==this.number-1?0:this.currentElement+1;
        this.lis[prev].classList.add("animateForward");
        show(this.lis[prev]);
        show(this.lis[this.currentElement]);
        clearTimeout(this.delayHideSetTimeOutControl);
        this.delayHideSetTimeOutControl=delayHide(this.lis[prev],1100);
        this.lis[this.currentElement].classList.add("active");
        this.lis[this.currentElement].classList.add("backward");
         if(this.auto)this.auto(this.intervalTime);        
    }
    removeClasses(){
        for(var i=0;i=this.number){
            this.currentElement=0;
        }
        this.removeClasses();
        var prev=this.currentElement==0?this.number-1:this.currentElement-1;
        this.lis[prev].classList.add("animateBack");
        show(this.lis[prev]);
        show(this.lis[this.currentElement]);
        clearTimeout(this.delayHideSetTimeOutControl);
        this.delayHideSetTimeOutControl=delayHide(this.lis[prev],1100);
        this.lis[this.currentElement].classList.add("active");
        this.lis[this.currentElement].classList.add("forward");
    }
    auto(ms){
        this.autoMode=true;
        this.intervalTime=ms;
        this.interval=setInterval(this._forWard.bind(this),ms);
    }
}
function delayHide(el,ms){
    return setTimeout(()=>el.style.display="none",ms);
}
function show(el){
    el.style.display=null;
}
function backWard(){
    slider.backWard();
}

function forWard(){
    slider.forWard();
}
```
The class is very simple and have three main methods named as auto, forWard and backWard. With one more method (forWard ) as a wrapper of forWard function . The method auto is for setting ‘keep playing’ mode slider ON.


https://mavrin.github.io/svgToPng

# How we were making export plugin or convert svg to png.

Sometimes we need to convert svg to png with help browser api. Unfortunately, browser don't have that magic api, which do it without different hack. What to do, if we want to achieve desire result?  ![](http://habrastorage.org/storage2/d36/922/412/d36922412b65ad20413ac591db392e09.png)
My first idea was make it with help canvas, which have method `toDataURL('image/png');`
So, I wrote simple script like [jsfiddle](http://jsfiddle.net/a9ude9p0/6/), [github](http://mavrin.github.io/svgToPng/fromImage.html):
```javascript
var html = document.querySelector("svg").parentNode.innerHTML;
var imgsrc = 'data:image/svg+xml;base64,' + btoa(html);
var canvas = document.querySelector("canvas"),
        context = canvas.getContext("2d");
canvas.setAttribute('width', 526);
canvas.setAttribute('height', 233);

var image = new Image;
image.src = imgsrc;
image.onload = function () {
    context.drawImage(image, 0, 0);
    var canvasdata = canvas.toDataURL("image/png");
    var a = document.createElement("a");
    a.textContent = "save";
    a.download = "export_" + Date.now() + ".png";
    a.href = canvasdata;
    document.body.appendChild(a);
    canvas.parentNode.removeChild(canvas);
};
```
The bottom line this script is simple, I convert svg to dataUri and load it from `<img src='dataUri' />`, draw image on canvas and make png. Seems that goal was achieved, and we can relax. This approach work in Chrome and Firefox, but if you run this script in IE, you get next error:
![secureError](http://habrastorage.org/files/1c6/74c/de8/1c674cde8f51425a82a653e55e86bc9e.png)
The reason this error is that image was loaded from no origin host. Unfortunately, we can't set `origin` for `dataUri`. Description these rules we can find here: https://html.spec.whatwg.org/multipage/scripting.html#security-with-canvas-elements. Of course we can proxy svg through server and this approach will work, but we want pure client solution.

Ok. Let's try other approach. I remembered great library [canvg](https://github.com/gabelerner/canvg). I draw svg on canvas with help this library, further I do as first approach. I get `toDataURL("image/png")`. It looks like [github](http://mavrin.github.io/svgToPng/useCanvg.html):
```javascript
var svg = document.querySelector('svg');
var canvas = document.createElement('canvas');
canvas.height = svg.getAttribute('height');
canvas.width = svg.getAttribute('width');
canvg(canvas, svg.parentNode.innerHTML.trim());
var dataURL = canvas.toDataURL('image/png');
var data = atob(dataURL.substring('data:image/png;base64,'.length)),
        asArray = new Uint8Array(data.length);

for (var i = 0, len = data.length; i < len; ++i) {
    asArray[i] = data.charCodeAt(i);
}

var blob = new Blob([asArray.buffer], {type: 'image/png'});
saveAs(blob, 'export_' + Date.now() + '.png');
```
It is worth noting, that i use [FileSaver](https://github.com/ChenWenBrian/FileSaver.js) as polyfill HTML5 W3C `saveAs()`.

And also one point for applying style from css i put css rules inline in svg.
[Result](http://mavrin.github.io/svgToPng/tauChartsExample.html). Just click on link `Save chart to png` and wait on that page.

That's all, we achieved our goal.

I hope, that article have been useful for you and save you time.

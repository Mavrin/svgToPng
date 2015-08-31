
https://mavrin.github.io/svgToPng

# How we made our export plugin and perform client-side, in-browser conversion of SVG's to PNG's.

Sometimes we need to convert SVG's to PNG's with the help of our browser. Unfortunately, browsers don't have a magic API that allows us to do this without some different hacks. So, what should we do if we need to do these conversions?  ![](http://habrastorage.org/storage2/d36/922/412/d36922412b65ad20413ac591db392e09.png)
My first idea was to do it with a little help from a canvas, which conveniently has the method `toDataURL('image/png');`
So, I wrote simple script like this: [jsfiddle](http://jsfiddle.net/a9ude9p0/6/), [github](http://mavrin.github.io/svgToPng/fromImage.html):
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
The premise behind this script is simple: I convert the SVG to a dataUri and load it from `<img src='dataUri' />`, draw the image on the canvas and finally make the PNG. Seems like we've achieved our goal and now we can relax. This approach works in Chrome and Firefox, but if you run this script in IE you'll get an error:
![secureError](http://habrastorage.org/files/1c6/74c/de8/1c674cde8f51425a82a653e55e86bc9e.png)
The reason for this error is that the image was loaded with no origin host. Unfortunately, we can't set `origin` for a `dataUri`. You can find a little list of various rules like this here: https://html.spec.whatwg.org/multipage/scripting.html#security-with-canvas-elements. Of course we can proxy the SVG through our server and this approach will work, but we want a pure client-side solution.

Ok. Let's try other approach. I remembered about this cool library, [canvg](https://github.com/gabelerner/canvg). I can draw my SVG on the canvas with a little help from this library and still stick with our first plan of attack. I get `toDataURL("image/png")`. This approach looks like this: [github](http://mavrin.github.io/svgToPng/useCanvg.html):
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
It is worth noting, that I use [FileSaver](https://github.com/ChenWenBrian/FileSaver.js) as a polyfill for W3C HTML5's `saveAs()`.

One side point that is definitely worth mentioning: to handle applying CSS styles, I put my CSS rules inline in my svg.
[Result](http://mavrin.github.io/svgToPng/tauChartsExample.html). Just click on the link `Save chart to png` and wait.

That's all, we achieved our goal. I hope that this article has been useful for you and will save you time.

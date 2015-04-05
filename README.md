
https://mavrin.github.io/svgToPng

# Convet svg to png

Иногда появляется необходимость сохранить svg в png средствами браузера. К сожалению, браузер не имеет волшебного api, который позволил бы это сделать без различных хаков. Что же делать, если все таки хочется добиться желаемого?  ![](http://habrastorage.org/storage2/d36/922/412/d36922412b65ad20413ac591db392e09.png)
Первая идея, которая мне пришла в голову, сделать это через canvas, который имеет метод `toDataURL('image/png');`
Итак, я написал простенький скрипт: [jsfiddle](http://jsfiddle.net/a9ude9p0/6/), [github](http://mavrin.github.io/svgToPng/fromImage.html):
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
Суть скрипта проста: я преобразовывал svg в dataUri, загружал его через image, рисовал картинку на canvas и превращал в png. Казалось, цель достигнута, и можно расслабится. Этот подход сработал в Firefox и Chrome, но открыв ~~во всеми нами любимом браузере~~ IE, я получил замечательную ошибку:
![secureError](http://habrastorage.org/files/1c6/74c/de8/1c674cde8f51425a82a653e55e86bc9e.png) 
Дело в том, что IE считает, что картинка загружена с другого хоста.  К сожалению, установить origin для dataUri не получится. Собственно, описание правил можно найти здесь:  https://html.spec.whatwg.org/multipage/scripting.html#security-with-canvas-elements. Можно было, конечно, проксировать svg через сервер, и тогда все бы сработало, но хотелось чисто клиентское решение.

И тут я вспомнил про замечательную библиотеку [canvg](https://github.com/gabelerner/canvg). С помощью этой библиотеки я рисую svg на canvas, а далее поступаю как в первом варианте: беру `toDataURL("image/png")`. Получился такой незамысловатый код: [github](http://mavrin.github.io/svgToPng/useCanvg.html):
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
Тут стоит сказать, что еще я использовал библиотеку [FileSaver](https://github.com/ChenWenBrian/FileSaver.js) для вызова диалогового окна сохранения.
Вот и все, мы добились желаемого результата.

Стоит отменить один нюанс - я задался вопросом сохранения svg в png, когда писал плагин для экспорта [tauCharts](http://www.taucharts.com/). Так как стили в svg задаются из внешнего файла, чтобы добиться максимально подобия с исходным svg, я вставляю inline style в svg. И получаем вот такой 
[результат](http://mavrin.github.io/svgToPng/tauChartsExample.html).

Надеюсь, статья окажется полезной для вас и сохранит ваше время.

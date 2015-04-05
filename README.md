
https://mavrin.github.io/svgToPng

# Convet svg to png

»ногда есть необходимость сохранить svg в png средствами браузера.   сожаление браузер не имеют волшебного api, который позволил бы это сделать без различных хаков. „то же делать, если все таки хочетс€ добитьс€ желаемого?  ![](http://habrastorage.org/storage2/d36/922/412/d36922412b65ad20413ac591db392e09.png)
ѕерва€ иде€, котора€ мне пришла в голову, сделать это через canvas, который имеет метод `toDataURL('image/png');`
» так € написал простенький скрипт [jsfiddle](http://jsfiddle.net/a9ude9p0/6/), [github](http://mavrin.github.io/svgToPng/fromImage.html):
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
—уть которого проста, € преобразовывал svg в dataUri, загружал его через image, рисовал картинку на canvas и превращал в png.  азалось цель достигнута, и можно расслабитс€. Ётот подход сработал в Firefox и chrome, но открыв ~~во всеми нами любимом браузере~~ IE, € получил замечательную ошибку:
![secureError](http://habrastorage.org/files/1c6/74c/de8/1c674cde8f51425a82a653e55e86bc9e.png) 
ƒело в том что IE считает, что картинка загружена с другого хоста.    сожалению установить origin дл€ dataUri не получитс€. —обственно описание правил https://html.spec.whatwg.org/multipage/scripting.html#security-with-canvas-elements. ћожно было конечно проксировать svg через сервер и тогда все бы сработало, но хотелось чисто клинское решение.
» тут € вспомнил про замечательную библиотеку [canvg](https://github.com/gabelerner/canvg). — помощью этой библиотеки € рисую svg на canvas, а далее поступаю к в первом случае беру `toDataURL("image/png")`. ѕолучилс€ такой не замысловатый код [github](http://mavrin.github.io/svgToPng/useCanvg.html):
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
“ут стоит сказать, что € еще использовал библиотеку [FileSaver](https://github.com/ChenWenBrian/FileSaver.js) дл€ вызова диалогового окна сохранени€.
¬от и все, мы добились желаемого результата.
—тоит отменить один нюанс, € задалс€ вопросом сохранени€ svg в png, когда писал плагин дл€ экспорта [tauCharts](http://www.taucharts.com/). “ак как стили в svg задаютс€ из внешнего файла, что бы добитьс€ максимально подоби€ с исходным svg, € вставл€ю inline style в svg. B получаем вот такой 
[результат](http://mavrin.github.io/svgToPng/tauChartsExample.html)

Ќадеюсь стать€ окажетс€ полезной дл€ вас и сохранит вам врем€.

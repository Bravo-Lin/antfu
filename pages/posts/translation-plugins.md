---
title: 「实现一个浏览器翻译插件」
date: 2022-05-10T10:40:00Z
lang: zh
duration: 15min
description: Develop translation plugins from scratch
hide: true
---

因为经常有看英文文档的需要，在阅读一大段解释其原理的过程中偶尔还是不得不借助一些工具。出于兴趣稍微学习了一下浏览器的插件开发，写了一个简单的划词翻译插件。

## 实现效果

> ctrl+alt 启用该插件，划词触发翻译。

![img](/images/translation.png)
![img](/images/translation2.png)

## 开发文档

浏览器拓展的开发实际上很好入门，完全可以跟着官方文档走：

> [Chrome Developer - Extension](https://developer.chrome.com/docs/extensions/mv3/getstarted)

> [Edege Developer - Extension](https://docs.microsoft.com/zh-cn/microsoft-edge/extensions-chromium/)

## 实现思路

我们实现的功能算是十分精简，因此整个项目并不复杂，我们需要做的事：

1. 添加元素，用于容纳译文，展示插件状态切换。
2. 监听鼠标，键盘事件，用于开启插件，获取选中文本。
3. 根据选中文本获取译文。

## 核心实现

添加翻译框元素，后续划词选中的译文将添加到该节点。

```js
const translator = document.createElement('div')
// 标识翻译框元素
translator.setAttribute('id', 'translatorArea')
translator.classList.add('baseStyle')
document.body.appendChild(translator)

···
//翻译框基础样式
const globalStyle = document.createElement("style");
globalStyle.innerHTML = `
.baseStyle {
  font-family:"Times New Roman";
  display: none;
  position:absolute;
  z-index:999999999;
  max-width: 30vw;
  line-height: 1.5em;
  border: 1px solid black;
  padding:10px;
  color: inherit;
  font-size: 15px;
  background: transparent;
  backdrop-filter: blur(5px);
  border-radius: 10px;
}
`;
```

监听鼠标按下/松开事件，分别进行处理：

1. 划词结束（鼠标左键松开事件）获取鼠标位置坐标作为翻译框的 position。
2. 获取选中文本的语言类型，根据类型进行原文语言类型选择译文。（当前规则为其他语言=>中文，中文=>英文）。
3. 获取译文，变更翻译框元素的 innerHtml。

```js
// 鼠标点击事件判断是否处于翻译框上
document.addEventListener("mousedown", (evt) => {
  if ((evt.target as HTMLElement).id !== "translatorArea") {
    translator.innerHTML = "";
    setStyle(translator, {
      display: "none",
    });
  }
});
···

//根据


//判断语言是否为中文（这里遇到日语会有 亚洲结构文字语言编码存在交集的问题，有时会误判为中文 －O－ ）
function isChinese(text: string): boolean {
  return /[\u4E00-\u9FA5\uF900-\uFA2D]/.test(text);
}
//原文为中文译文则为英文，原文非中文则译文为中文
function getLangOriginAndTarget(text: string): [SupportLang, SupportLang] {
  const o: SupportLang = isChinese(text) ? "zh-CN" : "auto";
  const t: SupportLang = o === "zh-CN" ? "en" : "zh-CN";
  return [o, t];
}
//获取原文语言和目标译文语言，传给谷歌翻译的服务端渲染的网站，在响应中通过元素类名查询获得内嵌的译文文本赋值给我们的翻译框元素并变更一下样式，OK，all done！。 
document.addEventListener("mouseup", (evt) => {
  if ((evt.target as HTMLElement).id !== "translatorArea") {
    chrome.runtime.sendMessage(
      {
        type: "check",
      },
      (resp) => {
        if (resp) {
            //获取选中文本
          const text = document.getSelection()?.toString();
          if (text?.trim()) {
            const [originLanguage, targetLanguage] = getLangOriginAndTarget(text,);
            chrome.runtime.sendMessage(
              {
                type: "fetch",
                url:
                  `https://translate.google.cn/m?ui=tob&hl=en&sl=${originLanguage}&tl=${targetLanguage}&q=${encodeURIComponent(text)
                  }`,
              },
              (resp) => {
                const elt = document.createElement("div");
                elt.innerHTML = resp;
                const result = (elt.querySelector(".result-container") as HTMLElement).innerText;
                setStyle(translator, {
                  display: "block",
                  top: `${evt.pageY}px`,
                  left: `${evt.pageX}px`,
                });
                translator.innerText = result;
              },
            );
          }
        }
      },
    );
  }
});
```

[快捷翻译@github]（<https://github.com/Bravo-Lin/Express-Translator>）

---
layout: post
title: Bind Handler Debugger Knockout
categories:
- JavaScript
- Knockout
tags:
- bibliotecas
- bind
- debug
- javascript
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _headspace_page_title: Bind Handler Debugger Knockout
  _headspace_description: Script para executar o debugger js durante o bind pelo Knockout de um elemento na view
author:
  login: maxcnunes
  email: maxcnunes@gmail.com
  display_name: maxcnunes
  first_name: ''
  last_name: ''
---

Knockout é uma ótima biblioteca javascript para aplicar o padrão MVVM no front-end. Mas dependendo do seu cenário, pode ser um pouco complicado para debuggar algum problema durante qualquer bind na view.<br />
Então para essas situações eu sempre utilizo um custom bind handler do knockout para invocar a pausa do js debugger durante o bind em tempo de execução:


```js
// Debugger Bind
//-------------------------
ko.bindingHandlers.debuggerBind = {
    init: function (element, valueAccessor, allBindingsAccessor) {
        debugger;
        var person = valueAccessor();
        console.debug('Init ~> Knockout bind debugger: ' + person.firstName());
    },
    update: function (element, valueAccessor) {
        debugger;
        var person = valueAccessor();
        console.debug('Update ~> Knockout bind debugger: ' + person.firstName());
    }
};
```

No bind eu adiciono o atributo **debuggerBind** e carrego a página no navegador com a ferramenta do desenvolvedor aberta. O resultado será o seguinte:


![knockout-debugger](/assets/knockout-debugger.png)

Debugger pausado durante bind:

![knockout-debugger2](/assets/knockout-debugger2.png)

[Veja o exemplo no JsFiddle](http://jsfiddle.net/maxcnunes/vYY7T/) (Execute com a ferramenta do desenvolvedor aberta - F12)

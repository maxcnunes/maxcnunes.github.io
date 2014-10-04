---
layout: post
title: JQuery UI dialog com eventos do ASP.NET
categories:
- .NET
- C#
- JavaScript
- JQuery
tags:
- c#
- javascript
- JQuery
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _aioseop_keywords: JQuery, UI, dialog, eventos, ASP.NET
  _aioseop_description: JQuery UI dialog com eventos do ASP.NET
  _aioseop_title: JQuery UI dialog com eventos do ASP.NET
  simplecatch-sidebarlayout: ''
author:
  login: maxcnunes
  email: maxcnunes@gmail.com
  display_name: maxcnunes
  first_name: ''
  last_name: ''
---

<img class="size-full wp-image-596 " title="jquery-ui-dialog-events-aspnet" src="/assets/jquery-ui-dialog-events-aspnet.png" alt="" width="262" height="147" />


Você começa um projeto novo e decidi aumentar a usabilidade, e nada melhor do que utilizar as várias funcionalidades que o plugin JQuery UI oferece. Dentro os vários, um bem legal é o dialog que é uma janela flutuante na tela.


Após escolher utilizar o dialog, quando finalmente chega o momento de testar as funcionalidades dos eventos de botões dentro do dialog, você percebe que após aberto, os eventos ASP.NET dentro dele deixam de funcionar. Isso porque quando o dialog é aberto, ele é movido para fora do form da página, perdendo assim as funcionalidades dos eventos.


<!--more-->


Para resolver este problema, basta mover a div do dialog para dentro do form novamente. Da seguinte forma:


```js
$("#dialog").dialog({
  height: 200,
  width: 400,
  modal: true,
  title: "Dialog",
  autoOpen: false,
  resizable: false,
  open: function (type, data) {
    //Move a div do dialog para dentro do form novamente
    $(this).parent().appendTo("form:first");
  }
});
```

Segue o código de uma tela completa, lembrando que ao utilizar um evento do ASP.NET é provocado um PostBack na tela, dessa forma para que a página toda não seja renderizada novamente e o dialog não seja fechado é necessário manter o conteúdo do dialog dentro de um UpdatePanel.


```html
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
  <title>Dialog UI with ASP.NET Events</title>
  <script src="Scripts/jquery-1.4.4.js" type="text/javascript"></script>
  <script src="Scripts/jquery-ui-1.8.19.js" type="text/javascript"></script>

  <link href="Content/themes/base/jquery-ui.css" rel="stylesheet" type="text/css" />
  <script type="text/javascript">
    $(function () {
      $("#dialog").dialog({
        height: 200,
        width: 400,
        modal: true,
        title: "Dialog",
        autoOpen: false,
        resizable: false,
        open: function (type, data) {
          //Move a div do dialog para dentro do form novamente
          $(this).parent().appendTo("form:first");
        }
      });

      $("#opener").click(function () {
        $("#dialog").dialog("open");
        return false;
      });
    });
  </script>
</head>
<body>

<form id="form1" runat="server">
  <div>
    <asp:scriptmanager id="ScriptManager1" runat="server">
    </asp:scriptmanager>
    <button id="opener">Open Dialog</button>

    <div id="dialog" title="Basic dialog">
      <asp:updatepanel id="UpdatePanel1" runat="server">
        <contenttemplate>
          Postback Test
          <asp:textbox id="tb_send" runat="server"></asp:textbox>
          <asp:button id="but_OK" runat="server" text="Send request"
            onclick="but_OK_Click" />

          <asp:label id="lbl_result" runat="server" forecolor="#ff0000"></asp:label>
        </contenttemplate>
      </asp:updatepanel>
    </div>
  </div>
</form>

</body>
</html>
```


E por último o evento do botão em c#:


```csharp
protected void but_OK_Click(object sender, EventArgs e)
{
    lbl_result.Text = "Response: " + tb_send.Text;
}
```


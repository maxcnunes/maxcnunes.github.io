---
layout: post
title: Funções básicas da API Google Maps
categories:
- JavaScript
tags:
- google map
- javascript
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  _aioseop_keywords: api, google, map, 3, bound, resize, rota, marcadores
  _aioseop_description: Tutorial funções básicas da API Google Maps. Carregar mapa
    pelo endereço, localização, adicionar marcadores, corrigir fronteira do mapa...
  _aioseop_title: Funções básicas da API Google Maps
  simplecatch-sidebarlayout: ''
author:
  login: maxcnunes
  email: maxcnunes@gmail.com
  display_name: maxcnunes
  first_name: ''
  last_name: ''
---

Com o google maps cada vez mais difundido entre a população. Pela facilidade na utilização do aplicativo e pelos benefícios providos pelo mesmo. Está cada vez mais comum, nos depararmos com sites e sistemas utilizando tais funcionalidades do aplicativo, para indicar a localização de empresas, rotas e outros pontos interessantes.

O Google disponibiliza uma API para que os desenvolvedores utilizem os recursos do GMap. Neste [link](http://code.google.com/intl/pt-BR/apis/maps/documentation/javascript/reference.html) você encontrará toda a documentação desta API.

Na demonstração a seguir será abordado funcionalidades simples da API do GMaps, como:


* Marcar um ponto no mapa pelo endereço
* Marcar um ponto no mapa pela localização (Latitude e Longitude)
* Criar descrição de rotas
* Criar eventos para receber o endereço atual do marcador ao arrastá-lo
* Customizar ícones dos marcadores com imagens diferentes do padrão
* Definir a fronteira e o zoom do mapa, de acordo com todos os pontos adicionados nele

Para você ter uma ideia de como ficará, segue um printscreen do resultado final.

![](/assets/gmap-exemplo.png)

Faça o download do demo: [https://github.com/maxcnunes/GoogleMapsAPI3](https://github.com/maxcnunes/GoogleMapsAPI3)

```js
/*
:::::: Variaveis globais
*/
var map;
var directionsPanel;
var directions;
var from;
var to;
var urlPrint;
var lat, lng;
var bounds;
var geocoder = new google.maps.Geocoder();
var directionsService = new google.maps.DirectionsService();

function CarregarMapa() {
    map = new google.maps.Map(document.getElementById('gmap'), {
        'zoom': 3,
        'center': new google.maps.LatLng(70, 0),
        'mapTypeId': google.maps.MapTypeId.ROADMAP
    });

    //Inicializa a o limite do mapa
    bounds = new google.maps.LatLngBounds();

    //Cria o marcador
    marker1 = new google.maps.Marker({
        map: map,
        position: new google.maps.LatLng(65, -10),
        draggable: true,
        title: 'Arraste-me!'
    });
    //Redimensiona o limite do mapa, de acordo com o novo ponto
    bounds.extend(new google.maps.LatLng(65, -10));

    //Carrega valores para a latitude e longitude
    lat = parseFloat($("#hdnLatitude").val());
    lng = parseFloat($("#hdnLongitude").val());

    marker2 = new google.maps.Marker({
        map: map,
        position: new google.maps.LatLng(lat, lng),
        draggable: true,
        icon: 'http://google-maps-icons.googlecode.com/files/airport.png'
    });
    //Redimensiona o limite do mapa, de acordo com o novo ponto
    bounds.extend(new google.maps.LatLng(lat, lng));

    //Cria um evento para o marcador, que será disparado ao arrastá-lo
    google.maps.event.addListener(marker1, "drag", function () {

        //Recupera a localizacao do marcador
        var latLng = marker1.getPosition();

        //Passa a localizacao do ponto ao arrastar, para inputs text e hidden
        $("#txtLatitude").val(latLng.lat());
        $("#txtLongitude").val(latLng.lng());
        $("#hdnLatitude").val(latLng.lat());
        $("#hdnLongitude").val(latLng.lng());
    });

    //Cria um evento para o marcador, que será disparado ao terminar arrastá-lo
    google.maps.event.addListener(marker1, "dragend", function () {

        geocodePosition(marker1.getPosition());
    });

    //Cria um evento para o marcador, que será disparado ao ser clicado
    google.maps.event.addListener(marker1, "click", function () {
        //Cria uma janela de mensagem para o marcador
        marker.openInfoWindowHtml("Arraste o ponto, para definir a localização");
    });

    //Redefine os limites
    map.fitBounds(bounds);
    //Redefine o zoom de acordo com o limite
    map.setZoom(map.getZoom());
    //Redefine o centro de acordo com o limite
    map.setCenter(bounds.getCenter());
}

function AdicionarLocalizacao() {
    var marker3 = new google.maps.Marker({
        map: map,
        position: new google.maps.LatLng(-70, -10),
        draggable: true,
        title: 'Arraste-me!',
        icon: 'http://mapicons.nicolasmollet.com/wp-content/uploads/mapicons/shape-default/color-ff8a22/shapecolor-white/shadow-1/border-color/symbolstyle-color/symbolshadowstyle-no/gradient-no/archery.png'
    });
    //Redimensiona o limite do mapa, de acordo com o novo ponto
    bounds.extend(new google.maps.LatLng(-70, -10));

    //Redefine os limites
    map.fitBounds(bounds);
    //Redefine o zoom de acordo com o limite
    map.setZoom(bounds.getZoom());
    //Redefine o centro de acordo com o limite
    map.setCenter(bounds.getCenter());
}

function CarregarPeloEndereco() {
    //Recupera o endereço no input text
    var endereco = $('#txtEndereco').val();

    //Carrega o endereco no mapa
    geocoder.geocode({
        address: endereco
    }, function (responses) {
        if (responses && responses.length > 0) {
            //Recupera a localiçao retornada pelo geocoder
            var place = responses[0];

            //Cria Marcador
            var marker4 = new google.maps.Marker({
                map: map,
                position: place.geometry.location,
                draggable: true,
                title: 'Arraste-me!',
                icon: 'http://mapicons.nicolasmollet.com/wp-content/uploads/mapicons/shape-default/color-ffc11f/shapecolor-color/shadow-1/border-dark/symbolstyle-white/symbolshadowstyle-dark/gradient-no/scoutgroup.png'
            });
            //Redimensiona o limite do mapa, de acordo com o novo ponto
            bounds.extend(place.geometry.location);

            //Atualiza valores nos inputs da pagina
            $('#txtEnderecoImovel').val(place.formatted_address);

            //Redefine os limites
            map.fitBounds(bounds);
            //Redefine o zoom de acordo com o limite
            map.setZoom(map.getZoom());
            //Redefine o centro de acordo com o limite
            map.setCenter(bounds.getCenter());
        } else {
            $('#txtEndereco').val('Nao pode determinar a localizacao.');
        }
    });
}

function CriarRota() {
    //Verifica se o input text esta preenchido
    if (Trim($("#ondeestou").val()) == "") {
        alert("Preencha a sua localizacao.");
        return;
    }
    //Limpa todas os marcadores
    //map.clearOverlays();

    //Define o painel que receberá a rota "escrita"
    directionsPanel = document.getElementById("rota_gmap");
    //Limpa o painel de qualquer html
    directionsPanel.innerHTML = "";

    //Recupera o endereco do input text
    from = $("#ondeestou").val();

    var directionsDisplay = new google.maps.DirectionsRenderer({
        'map': map,
        'preserveViewport': true,
        'draggable': true
    });
    directionsDisplay.setPanel(directionsPanel);

    var request = {
        origin: from,
        destination: new google.maps.LatLng(lat, lng),
        travelMode: google.maps.DirectionsTravelMode.DRIVING
    };
    directionsService.route(request, function (response, status) {
        if (status == google.maps.DirectionsStatus.OK) {
            directionsDisplay.setDirections(response);
        }
    });
}

function geocodePosition(pos) {
    geocoder.geocode({
        latLng: pos
    }, function (responses) {
        if (responses && responses.length > 0) {
            $('#txtEndereco').val(responses[0].formatted_address);
        } else {
            $('#txtEndereco').val('Nao pode determinar a localizacao.');
        }
    });
}

/*
::::: SCRIPTS DE SUPORTE
*/
function Trim(str) {
    return str.replace(/^s+|s+$/g, "");
}

function Imprimir() {
    var disp_setting = "toolbar=yes,location=no,directories=yes,menubar=yes,";
    disp_setting += "scrollbars=yes,width=643, height=600, left=100, top=25";
    var content_vlue = document.getElementById("rota_gmap").innerHTML;
    var docprint = window.open("", "", disp_setting);
    docprint.document.open();
    docprint.document.write("<html><head><title>Como chegar</title>");
    docprint.document.write("<style type="text/css">body{ font-family:Tahoma;font-size:8pt; margin:10px; }</style>");
    docprint.document.write("</head><body><center>");
    docprint.document.write("<h1>Como chegar</h2>");
    docprint.document.write("<div  style="width: 603px; margin-left:2px;">");
    docprint.document.write(content_vlue);
    docprint.document.write("</div>");
    docprint.document.write("<script type="text/javascript" language="javascript">window.print();</script></body></html>");
    docprint.document.close();
    docprint.focus();
}
```

```html
<html>
<head>
    <title>Google Maps API</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <!-- :: CSS - Page Layout :: -->
    <link href="css/common.css" rel="stylesheet" type="text/css" />
    <!-- :: JS - JQuery :: -->
    <script type="text/javascript" src="http://code.jquery.com/jquery-1.7.1.min.js"></script>
    <!-- :: JS - Google Maps API :: -->
    <script type="text/javascript" src="http://maps.google.com/maps/api/js?sensor=false"></script>
    <!-- :: JS - GMaps Functions :: -->
    <script type="text/javascript" src="js/js-gmaps-functions.js"></script>
    <script type="text/javascript" language="javascript">
        $(function () {
            //Inicializa o mapa
            CarregarMapa();
        });
    </script>
</head>
<body>
    <div id="wrapper">
        <!--Valores pré-definidos para o mapa-->
        <input id="hdnLatitude" runat="server" type="hidden" value="-23.292757" />
        <input id="hdnLongitude" runat="server" type="hidden" value="-51.169424" />
        <div>
            <div class="box_left">
                <label>
                    Endereço:</label>
                <input type="text" id="txtEndereco" value="Av. Tiradentes 1241, Londrina PR" size="54" />
                <!--Botao busca o mapa pelo endereço-->
                <a class="button" href="javascript:CarregarPeloEndereco();" title="Carregar a partir do endereço">
                    <img src="img/search.png" alt="Buscar" />
                </a>
                <div class="box_middle">
                    <label>
                        Latitude:</label>
                    <input type="text" id="txtLatitude" value="-23.292757" />
                </div>
                <div class="box_middle">
                    <label>
                        Longitude:</label>
                    <input type="text" id="txtLongitude" value="-51.169424" />
                    <!--Botao busca o mapa pela localização (Latitude e Longitude)-->
                    <a class="button" href="javascript:CarregarMapa();" title="Carregar a partir da longitude e latitude">
                        <img src="img/search.png" alt="Buscar" />
                    </a>
                </div>
                <div>
                    <label>
                        Adicionar mais 1 localização
                        <!--Botao adiciona uma nova localização no mapa-->
                        <a class="button" href="javascript:AdicionarLocalizacao();">
                            <img src="img/add.png" alt="Add" />
                        </a>
                    </label>
                </div>
            </div>
            <div class="box_right">
                <label>
                    Como chegar?</label>
                <input type="text" id="ondeestou" size="36" />
                <!--Botao criar rota-->
                <a href="javascript:CriarRota();" title="Traçar Rota" class="button">
                    <img src="img/go.png" alt="Ir" /></a>
                <br />
                <span style="font-size: 8pt; color: #999;">Ex: Rua Numero, Cidade, Estado</span>
                <br />
                <br />
                <!--Botao imprimir-->
                <a href="javascript:Imprimir();" class="button">
                    <img src="img/print.png" alt="Imprimir" /><span>Imprimir rota</span> </a>
            </div>
        </div>
        <div>
            <div class="box_left">
                <!--Div que armazenará o mapa-->
                <div id="gmap">
                </div>
            </div>
            <div class="box_right">
                <!--Div que armazenará a descrição da rota-->
                <div id="rota_gmap">
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```


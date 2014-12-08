---
layout: post
title: Libxmljs - Finding XML elements with namespaces using xpath
categories:
- Js
tags:
- js
- libxmljs
- xpath
- namespace
- xml
---

Today I spent a stupid time trying to parse a XML input to JS. Even using a great module [libxmljs](https://github.com/polotek/libxmljs) I was having problems to read any element that was not inside of client element (xpath: `//client`).

```xml
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <soap:Body>
    <Response xmlns="http://tempuri.org/">
      <Result>
        <client xmlns="">
          <msg>SEARCH OK.</msg>
          <code>0</code>
        </client>
      </Result>
    </Response>
  </soap:Body>
</soap:Envelope>
```

Why I could not get any of these elements?

* **soap:Envelope** - xpath: `//soap:Envelope`
* **soap:Body** - xpath: `//soap:Body`
* **Response** - xpath: `//Response`
* **Result** - xpath: `//Result`

This question forced me study a bit more about xml structure. In a first moment I didn't want to study it because XML is too boring. But I was spending too much time just guessing and searching a solved solution for my problem.

Reading [XML Namespaces and How They Affect XPath and XSLT](http://msdn.microsoft.com/en-us/library/ms950779.aspx) I could finally understand why I could get only elements inside of **client**.

Summarizing that article there are three types of namespaces:


### Prefix Namespaces

A namespace is composed of a name and a URI specification. The scope of the prefix-namespace mapping is that of the element that the namespace declaration occurs on as well as all its children.

```xml
<bk:bookstore xmlns:bk="urn:xmlns:25hoursaday-com:bookstore">
 <bk:book>
    <bk:title>Lord of the Rings</bk:title>
    <bk:author>J.R.R. Tolkien</bk:author>
    <inv:inventory status="in-stock" isbn="0345340426"
        xmlns:inv="urn:xmlns:25hoursaday-com:inventory-tracking" />
 </bk:book>
</bk:bookstore>
```

### Default Namespaces

Since use prefix namespace for all element is too verbose. Any element without a prefix namespaces is associated to the default namespace.

```xml
<bookstore xmlns="urn:xmlns:25hoursaday-com:bookstore">
 <book>
    <title>Lord of the Rings</title>
    <author>J.R.R. Tolkien</author>
    <inv:inventory status="in-stock" isbn="0345340426"
        xmlns:inv="urn:xmlns:25hoursaday-com:inventory-tracking" />
 </book>
</bookstore>
```

### Undeclared Namespaces

If an element has blank namespace (`xmlns=""`) then it is a **undeclared** namespace and any children element without a prefix namespace is associated to that undeclared namespace.

```xml
<bookstore xmlns="urn:xmlns:25hoursaday-com:bookstore">
 <book xmlns="">
    <title>Lord of the Rings</title>
    <author>J.R.R. Tolkien</author>
    <inv:inventory status="in-stock" isbn="0345340426"
        xmlns:inv="urn:xmlns:25hoursaday-com:inventory-tracking" />
 </book>
</bookstore>
```


Understanding these namespaces helped me finally solve my problem. The reason I could only get the **client** element and its children was because it was using an undeclared namespace. Using **libxmljs** is necessary specify the namespace on reading an element with namespace. Which means I was not passing the namespace for those elements with namespace and their children.

This is simple example to clarify the final solution:

{% gist maxcnunes/36dc98c0be4e00ad4c6c %}


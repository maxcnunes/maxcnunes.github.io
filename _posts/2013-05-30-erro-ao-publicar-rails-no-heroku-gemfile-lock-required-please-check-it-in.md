---
layout: post
title: Erro ao publicar Rails no Heroku - "Gemfile.lock required. Please check it in."
categories:
- Rails
- Ruby
tags:
- deploy
- heroku
- rails
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _aioseop_keywords: heroku, rails, gemfile.lock, required, error
  _aioseop_description: Solução para o erro que ocorre ao publicar Rails no Heroku "Gemfile.lock required. Please check it in."
  _aioseop_title: Erro ao publicar Rails no Heroku
  simplecatch-sidebarlayout: ''
author:
  login: maxcnunes
  email: maxcnunes@gmail.com
  display_name: maxcnunes
  first_name: ''
  last_name: ''
---

Passei um bom tempo quebrando a cabeça por causa desse problema a baixo que ocorria toda vez que eu tentava publicar o Rails no Heroku:

{% gist maxcnunes/5675847 %}

No final das contas a solução foi simples, mas como pode ser que ocorra com alguém mais, então esta foi a minha solução:

- Remova o Gemfile.lock do `.gitignore`

- Execute esse comando para remover a pasta de bundle local, executar o bundle localmente, adicionar o Gemfile.lock e commitar

```bash
rm -rf .bundle && bundle install && git add Gemfile.lock && git commit -m "Added Gemfile.lock"
```

- Execute esse comando para publicar no heroku (considerando que você esteja trabalhando com github simultaneamente).

```bash
git push heroku branch_do_github:master
```

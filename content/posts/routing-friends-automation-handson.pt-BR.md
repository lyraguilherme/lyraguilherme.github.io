---
title: 'Routing Friends - Network Automation'
date: 2024-11-21T18:50:09-03:00
draft: false
ShowToC: true
TocOpen: true
cover:
    image: /static/images/routing-friends/routing-friends.png
    alt: 'Routing Friends - Network Automation'
    caption: 'Routing Friends - Network Automation'
---

Estamos de volta com mais um episódio emocionante do Routing Friends! Desta vez, vamos conversar sobre automação de redes e trabalhar em um laboratório prático com ferramentas/frameworks específicos para realizar operações com automação!

Link para o vídeo:

[Desvendando a Automação de Redes: Lab Hands-On com Guilherme Lyra, CCIE #66666 | Episódio 167 ](https://www.youtube.com/watch?v=V8hF8toSAJ4)


## Guia de utilização do laboratório

### 1. Preparando seu host de automação
O primeiro passo é organizar o seu próprio host de automação, que nada mais é do que o computador onde você vai executar os scripts. O único ponto que deve ser observado é que esse computador precisa ter acesso aos equipamentos de rede do seu laboratório. No exemplo que eu apresentei, o meu "automation host" estava na mesma rede de gerência dos equipamentos de rede do laboratório, com acesso direto aos equipamentos.

Recomendo utilizar alguma distribuição Linux para isso, mas pode ser utilizado Mac ou Windows também.

Aplicações necessárias no seu host de automação:

- Python
- Ansible


### 2. Acesso aos scripts de exemplo

Acesse ou faça um clone do repositório abaixo. Nele, você encontrará a apresentação, topologia e scripts que foram utilizados durante o **Routing Friends - Lab Hands-on** realizado em 21 de novembro de 2024:
[lyraguilherme on GitHub](https://github.com/lyraguilherme/LabHandsOn-Nov2024/)
ou
```shell
git clone https://github.com/lyraguilherme/LabHandsOn-Nov2024/
```


### 3. Preparando o venv e dependências

Para utilizar os scripts, você deve preferencialmente criar um **venv**:
```shell
python -m venv venv
```

Ativar o venv:
```shell
source venv/bin/activate
```

Em seguida, instalar as dependências com o comando:
```shell
pip install -r requirements.txt
```


### 4. Definindo as credenciais como variáveis do SO

Após instalar todas as dependências, defina as credênciais em variáveis do seu SO. No meu caso, como mencionei, meu host de automação roda Linux então o procedimento é o seguinte:

```shell
export LAB_USERNAME="meu_usuario"
export LAB_PASSWORD="minha_senha"
```

A partir desse ponto, você conseguirá utilizar os scripts que compartilhei.
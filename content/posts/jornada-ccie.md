---
title: 'Minha Jornada ao CCIE'
date: 2024-02-24T14:07:09-03:00
draft: false
cover:
    image: /images/ccie/CCIE-66666.jpeg
    alt: 'Guilherme Lyra, CCIE #66666'
    caption: 'Guilherme Lyra, CCIE #66666'
---

Oficialmente comecei minha carreira em TI em 2006, mas, meus primeiros passos com redes ocorreram alguns anos antes disso, entre 2002 e 2003.

Em 2006 comecei como estagiário no suporte a usuários em um órgão do governo. Comecei formatando computadores e progredi rapidamente até um ponto em que lidava com toda a infraestrutrua de rede incluindo switches, firewalls, proxy, file servers, AD, etc.

Após finalizar esse estágio, fui para outra empresa, novamente como estagiário, onde passei mais 2 anos antes de ser oficialmente contratado. No total, trabalhei lá por quase 10 anos e tive a oportunidade de aprender com profissionais excepcionais. Enquanto trabalhava lá, obtive a certificação **CCNA** em **2008**, depois a certificação **CCDA** em **2013** e concluí a trilha da certificação **CCNP Routing & Switching** em **2017**.

Logo depois de finalizar a última prova da track e obter a certificação CCNP R&S, comprei alguns livros e comecei a estudar o conteúdo da INE e da extinta IPExpert. Quando iniciei os estudos, a prova ainda era a CCIE Routing & Switching v5.0. Em determinado momento o exame foi atualizado para a versão CCIE Routing & Switching v5.1 e, pouco tempo depois, eu troquei de emprego. Era Fevereiro de 2018 e eu passei a atuar como Líder Técnico de um time composto por engenheiros de redes e segurança. Devido ao grande volume de trabalho, infelizmente tive que reduzir o ritmo dos estudos. Não cheguei a parar completamente de estudar, mas, já estava em um ritmo bastante devagar.

Ainda na mesma empresa, no final de 2019 passei a integrar uma equipe recém-criada que chamamos de equipe de Arquitetura de Soluções. Eu passei a ser o responsável não apenas pela implementação, mas também pelo Design de todos os projetos de redes da empresa. Ainda em 2019, precisei renovar minhas certificações e foi assim obtive a certificação Cisco CCDP, que na época ainda era uma certificação a parte e posteriormente se transformou em uma especialização, atualmente chamada Cisco Certified Specialist - Enterprise Design.

Em 2020, com o início da pandemia e o trabalho remoto, consegui reorganizar minha rotina de forma a retomar os estudos para o CCIE com foco total. A prova já estava em uma terceira versão desde que eu havia iniciado os estudos, não se chamava mais CCIE Routing & Switching e sim CCIE Enterprise Infrastructure v1.0.

Então, resumindo, comecei oficialmente os estudos para o CCIE em 2017, ao concluir o CCNP, época em que a prova ainda era a CCIE R&S v5.0. Devido a questões de rotina, diminuí o ritmo de estudos e só consegui retomar em 2020. Basicamente, do início de 2020 até dezembro de 2022 (data em que realizei o lab), o meu foco foi exclusivamente em estudar. Na sequência desse post entrarei em maiores detalhes sobre estratégias, horas de estudos, entre outros.

---

> Todas as informações a respeito da prova que você vai encontrar aqui são públicas e estão disponíveis no site da Cisco.

---

O primeiro ponto a entender é que para que você possa agendar o CCIE LAB você precisa ter sido aprovado na prova 350-401 ENCOR dentro do período de 3 anos. Se você estiver elegível, basta agendar o lab através da ferramenta de agendamento: https://ccie.cloudapps.cisco.com/CCIE/Schedule_Lab/CCIEOnline/CCIEOnline

Muita gente não sabe, mas, ao contrário de provas como o CCNA e CCNP, que são realizados em centros credenciados de treinamento, o CCIE LAB é realizado de forma presencial em algum laboratório da própria Cisco, nas localidades abaixo:

![CCIE Lab Locations](/images/ccie/ccie_lab_locations.png)
*Fonte: https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/expert/ccie-lab-exam-locations.html*

Perceba na tabela acima que existem os chamados Mobile Labs que, como o nome já diz, são labs temporários que ocorrem em determinados locais. No momento em que escrevo esse post (janeiro de 2024), vi que existem algumas datas em que o lab estará no Brasil, em abril de 2024. Porém, consultando pela ferramenta de agendamento, não encontrei vagas disponíveis. O lado ruim dos Mobile Labs é esse: as vagas costumam fechar rapidamente. Você pode consultar as datas dos Mobile Labs aqui: https://learningnetwork.cisco.com/s/mobile-lab-scheduler

Por uma questão de logística, eu optei por fazer o lab em um local fixo e agendei minha prova no temido Building 5 da Cisco na cidade de Richardson, Texas.

![Cisco Building 5, Richardson TX](/images/ccie/Richardson_Building_5.jpg)
*Cisco Building 5 - Richardson, Texas - Dezembro de 2022*

Fazendo dessa forma, não tive nenhuma dificuldade para conseguir disponibilidade de agenda para o LAB. O que precisa ser considerado aqui são os custos de viagem. Coloque todos os itens na balança e decida como fica mais adequado para o seu caso.

Continuando, o CCIE LAB tem duração de 8 horas e está dividido em 2 módulos:

![CCIE Lab Modules](/images/ccie/labexam-modules.png)
*CCIE Lab Modules*

Para ser aprovado, você precisa atingir um score mínimo em cada um dos módulos e também um score mínimo final (pass score). Não adianta tirar o score máximo em um módulo e ficar abaixo do mínimo no outro módulo: nessa condição, você não vai ser aprovado.

Os valores de scores mínimos não são divulgados. O importante é entender que esse é o formato de pontuação da prova:

![CCIE Lab Modules](/images/ccie/labexam-scoreevaluation.png)
*CCIE Score Evaluation*

Outra detalhe crítico é que não existe score parcial. Ou seja, digamos que em determinada task você configurou tudo corretamente mas usou um nome de ACL diferente do que a questão pedia. Nesse caso, você não terá nenhum ponto naquela questão. Esse é um dos fatores que torna a prova tão difícil: você precisa fazer 100% da questão para ter a pontuação. Atenção aos detalhes é algo fundamental.

## Module 1: Design (3 hours)

O objetivo deste módulo é medir a capacidade de criar, analisar, validar e otimizar projetos de rede, que é a base para todas as atividades de implantação. Os candidatos precisarão:

* Compreender as capacidades de diferentes tecnologias, soluções e serviços.
* Traduzir os requisitos do cliente em soluções.
* Avaliar a prontidão da infraestrutura para apoiar as soluções propostas.

O módulo é baseado em cenários, sem acesso a nenhum dispositivo. Os candidatos receberão um conjunto de documentação necessária para discernir antes de responder as questões.

Exemplos de documentação incluem threads de e-mail, design de alto nível, diagramas de topologia de rede, requisitos e restrições do cliente, etc. Exemplos de questões incluem arrastar e soltar (drag & drop), resposta única de múltipla escolha, múltipla escolha, etc. Durante este módulo, a navegação para trás será desativada. Como tal, os candidatos não terão visibilidade total sobre todas as questões deste módulo. Os valores de pontos associados a cada item não são exibidos neste módulo.

## Module 2: Deploy, Operate and Optimize (5 hours)

Neste módulo, os candidatos irão implantar, operar e otimizar tecnologias e soluções de rede.

### Deploy ###
Os candidatos construirão a rede de acordo com as especificações do projeto, requisitos e restrições do cliente. Todas as etapas necessárias para uma implementação de rede bem-sucedida serão abordadas, incluindo configuração, integração e solução de problemas no comissionamento de tecnologias e soluções, conforme os tópicos do exame.

### Operate and Optimize ###
Os candidatos irão operar e otimizar tecnologias e soluções de rede. Isso inclui monitorar a integridade e o desempenho da rede, configurar a rede para melhorar a qualidade do serviço, reduzir interrupções, mitigar interrupções, reduzir custos operacionais e manter alta disponibilidade, confiabilidade e segurança, bem como diagnosticar possíveis problemas e ajustar configurações para se alinhar às mudanças. objetivos de negócios e/ou requisitos técnicos.Este módulo fornece uma configuração muito próxima de um ambiente de rede de produção real e consistirá em itens práticos (acesso ao dispositivo) e também em itens baseados na Web. Sempre que possível, um ambiente de laboratório virtualizado será utilizado neste módulo.

Durante este módulo a navegação para trás será habilitada. Os candidatos terão total visibilidade de todas as questões deste módulo. Os valores de pontos associados a cada item são exibidos neste módulo.

Fonte: https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format


--- 

## Primeiros passos ##

Baixe a planilha CCIE Enterprise Infrastructure Learning Matrix no link abaixo. Essa planilha foi criada e é mantida pela própria Cisco. Nela, você vai encontrar todos os tópicos da prova com algumas indicações de materiais de estudo. 

> No link a seguir, no final da página, em Resources, tem um link para a CCIE Enterprise Infrastructure Learning Matrix: https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/expert/ccie-enterprise-infrastructure.html#~exams

Na Perceba que a primeira coluna da planilha (chamada "Level") é justamente para que você atribua uma nota de 1 a 5 para o seu conhecimento em cada um desses tópicos. A planilha vem preenchida, mas, altere de acordo com o seu nível de conhecimento. Seja sincero, pois é assim que você saberá onde precisa dar mais ou menos foco nos estudos. Revise periodicamente essa planilha e atualize seu nível a medida que vai evoluindo.

## Dicas gerais ##

1. Aprenda inglês. Não precisa ser fluente no idioma, mas, a prova é toda em inglês e você precisa ter a capacidade de interpretar todos os detalhes possíveis do que está sendo solicitado, identificar pontos de atenção, "pegadinhas", etc.

2. Não acredite em receitas prontas. Não adianta ficar estudando X horas por dia sem ter uma estratégia e sem saber onde você precisa focar. Você só estará perdendo tempo.

3. A planilha CCIE Enterprise Infrastructure Learning Matrix será sua maior aliada nessa jornada. Utilize-a para encontrar seus pontos fracos e saber quais conteúdos você precisa estudar mais ou menos. Volte e leia novamente o que eu escrevi no item "primeiros passos".

4. Comece estudando poucas horas por dia e aumente de forma gradual o seu tempo de estudos. Não adianta fazer um esforço enorme no início e desistir no meio da jornada. Leia novamente o item 4.

5. No início dos estudos, se concentre em tópicos específicos e crie topologias simples para estudar. Evite de começar a jornada trabalhando com uma topologia muito grande. Qual o objetivo de trabalhar com 2 ou 3 protocolos ao mesmo tempo se você não entende nenhum deles de forma profunda? Você precisará disso na fase final, quando estiver perto de agendar o lab, mas há um caminho longo a ser percorrido até chegar lá

6. Enquanto estiver estudando teoria, não precisa pegar o livro e ler do inicio ao fim. Fazendo isso você provavelmente vai esquecer a maior parte do conteúdo. Procure focar em tecnologias/protocolos individualmente.

7. Não se frustre quando precisar ler e reler o mesmo assunto inúmeras vezes. Isso é parte do processo de aprendizado.

8. Eu particularmente gosto de consultar a teoria enquanto estou fazendo alguma configuração específica. Os livros em formato digital ajudam bastante nisso.

9. Entenda como determinado comando ou funcionalidade vai se refletir a nível de protocolo. Crie o hábito de analisar capturas de pacotes.

10. Ainda sobre capturas de pacotes, analise todas as camadas de forma aprofundada. Entenda o que muda ao realizar ou modificar determinada configuração, quais bits ou atributos são alterados, como a informação é propagada, etc.

11. Procure ler sobre um determinado assunto em diversas fontes diferentes. Ao estudar determinado tópico, revise as documentações da Cisco e também livros de diferentes autores. Leia novamente o item 10.

12. Estude sobre Design de redes. Normalmente existem vários caminhos para se chegar até o mesmo resultado. Por que implementar algo de um jeito e não de outro? Quais serão as diferenças? Qual delas atende de forma mais adequada o seu objetivo?

13. Na reta final dos estudos para o Lab, considero que aproximadamente 3 meses antes da sua data do lab, nesse ponto sim você deve conseguir estudar pelo menos 8 horas por dia. Nessa fase você deve estar focando em configuração e troubleshooting de topologias complexas. Lembre-se que a prova terá duração de 8 horas. Crie uma estratégia para conseguir manter a concentração e o foco em condições de stress e cansaço.

## Livros ##

Alguns dos que usei e considero bastante relevantes:

* CCIE Routing & Switching v5.0, Volume 1, 5th Edition [Kocharians & Paluch]
* CCIE Routing & Switching v5.0, Volume 2, 5th Edition [Kocharians & Paluch]
* Definitive MPLS Network Designs [Guichard]
* IP Multicast, Volume I [Loveless, Blair, Durai]
* IPv6 Fundamentals, 2nd Edition [Graziani]
* MPLS Fundamentals [De Ghein]
* Optimal Routing Design [White, Slice, Rentana]
* Routing TCP/IP, Volume 1, 2nd Edition [Doyle & Carrol]
* Routing TCP/IP, Volume 2, 2nd Edition [Doyle]
* Troubleshooting BGP [Edgeworth, Jain]

## Routing & Switching ##

* Lembre-se que os protocolos tradicionais ainda correspondem a uma grande parte da prova.
* Para os meus estudos de R&S usei os materiais antigos da INE, tanto os vídeos quanto o workbook da versão antiga da prova (CCIE R&S v5.0). Na minha opinião, o conteúdo deles é imbatível para a parte de R&S. Continuo utilizando até hoje as topologias do workbook para estudar ou simular algo.
* Para R&S, também utilizei materiais antigos da extinta IPExpert.
* Para laboratórios de R&S, usei o EVE-NG e Cisco CML.
* Procure estudar utilizando as mesmas versões de software que serão utilizadas na prova. Essas informações podem ser encontradas aqui: https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format na lista Equipment and Software você encontrará um PDF com as imagens e respectivas versões utilizadas na prova: CCIE Enterprise Infrastructure (v1.1) Equipment and Software List
* Não me peça para enviar alguma imagem para você. Caso não tenha acesso as imagens, você pode utiliza-las de forma legal através do Cisco CML.

## SD-WAN ##

* Comece pelos Configuration Guides e Design Guides da Cisco.
* Há bastante conteúdo em apresentações do Cisco, disponíveis em ciscolive.com
* Eu geralmente recomendo que a pessoa estude todo o material em inglês, mas, se estiver começando a estudar sobre SD-WAN, recomendo essa apresentação que eu fiz a convite da Comunidade Cisco: https://guilhermelyra.com/cisco-sd-wan-live-event/
* Revise com atenção os tópicos cobrados na CCIE Enterprise Infrastructure Learning Matrix. Não perca tempo estudando features que não serão cobradas no lab.
* Recomendo o conteúdo da INE (atualizado para CCIE Enterprise v1.0). Dentro da plataforma da INE, após concluir determinado módulo, geralmente tem alguns labs que eles disponibilizam para reforçar o conteúdo.
* Para laboratórios, caso tenha hardware disponível, pode utilizar o próprio VMware para virtualizar os componentes de SD-WAN. Lembre-se de utilizar as mesmas versões de software que caem na prova. Essas informações podem ser encontradas aqui: https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format na lista Equipment and Software você encontrará um PDF com as imagens e respectivas versões utilizadas na prova: https://learningcontent.cisco.com/documents/marketing/exam-topics/CCIEEIv1.1-equipment_and_SW.pdf
* O problema para rodar SD-WAN "dentro de casa" é que você vai precisar de uma máquina robusta. Caso não possua em casa ou no trabalho, busque alguma alternativa para alugar online um laboratório que possua essas imagens e recursos suficientes. Uma alternativa é alugar algumas sessões no ambiente da própria Cisco, chamado CCIE Enterprise Infrastructure Practice Labs: https://learningnetwork.cisco.com/s/article/ccie-enterprise-infrastructure-practice-labs

## DNA Center, SD-Access e ISE ##

* No meu caso, eu já tinha experiência com essas soluções em ambientes de produção. Fui responsável por implementar projetos de SD-Access e isso ajudou muito na hora da prova.
* Para laboratórios, não tem como fugir: você vai precisar de acesso a um DNA Center e switches da linha Catalyst 9000. No meu caso, usei diversas sessões no CCIE Practice Labs (CCIE Enterprise Infrastructure Practice Labs). O acesso ao lab tem que ser agendado previamente. Além disso, o valor é cobrado em dólar e se não me engano tem um limite de 4h por sessão. Gastei bastante dinheiro aqui, mas vale muito a pena.

## Automação ##

* **Python:** uma máquina rodando Linux (pode ser uma VM) e com conectividade a alguma topologia com routers rodando IOS XE. Se você não tem conhecimento nenhum em Python, comece pelo básico antes de sair interagindo com equipamentos.
* **EEM Applets e Guestshell:** sugiro utilizar algum router virtual rodando IOS XE. Pode ser CSR 1000v ou Catalyst 8000v.
* **APIs do vManage e DNA Center:**
    * **vManage:** é possível virtualizar "dentro de casa" ou usar algum lab do **DevNet Sandbox** da Cisco. 
    * **DNA Center:** falando especificamente das APIs, recomendo algum DevNet Sandbox da Cisco, onde você consegue utilizar a ferramenta sem ter que pagar. 
    * Os labs do **DevNet Sandbox** costumam ser read-only, então, caso queira manipular alguma configuração, sugiro que agende uma sessão no **CCIE Enterprise Infrastructure Practice Labs:** https://learningnetwork.cisco.com/s/article/ccie-enterprise-infrastructure-practice-labs
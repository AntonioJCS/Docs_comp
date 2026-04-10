## 📘 Visão Geral da Demanda

Esta entrega tem como objetivo analisar os **transbordos entre sistemas** no fluxo jurídico, com foco em responder se um transbordo foi **devido**, **indevido** ou **inconclusivo**, e, quando possível, apontar os elementos que sustentam essa classificação.

A necessidade de negócio por trás dessa análise não é apenas medir volume de transbordo. O ponto central é entender a **qualidade da passagem de casos entre sistemas**, identificar se o EX3 está deixando de tratar casos que poderia tratar e, ao mesmo tempo, distinguir situações em que o transbordo realmente era necessário por limitação funcional, exceção operacional ou continuidade natural do processo.

Em termos práticos, esta análise ajuda a responder perguntas como: quais transbordos foram corretos, quais poderiam ter sido evitados, quais casos ainda não podem ser avaliados com segurança e quais padrões observados podem orientar melhorias no processo, na operação e até na evolução funcional dos sistemas.

---

## 🧩 Contexto do Processo e dos Sistemas

O fluxo analisado envolve três sistemas que atuam de forma complementar ao longo da jornada do processo jurídico.

O **BJ** é o sistema de entrada. É nele que o processo nasce e onde a **pasta** é criada. Essa pasta funciona como o identificador transversal entre os sistemas e será a principal referência para integração analítica. Embora cada sistema possua suas próprias chaves internas, a pasta é o elo comum entre eles.

O **EX3** é o sistema responsável pelo tratamento dos **subsídios padronizados**. É nele que ocorre a captura de provas, a consolidação do laudo e a definição da estratégia para os casos que seguem o fluxo esperado e previsto para esse tipo de tratamento.

O **ONE** é o sistema que recebe os casos após a passagem pelo EX3. Isso pode acontecer de duas formas. A primeira é quando o caso foi tratado normalmente no EX3 e segue seu fluxo natural para as próximas etapas. A segunda é quando o caso entra no EX3, mas durante o tratamento é identificada alguma condição que impede sua conclusão naquele ambiente, fazendo com que ele seja transbordado para continuidade ou tratamento de exceção no ONE.

Nesse contexto, o conceito de **transbordo** precisa ser entendido de forma ampla. Todo caso que sai do EX3 em direção ao ONE é um transbordo. O que muda não é o fato de ter ocorrido o transbordo, mas sim o seu mérito. Em outras palavras, o objetivo da análise não é descobrir se houve transbordo, porque isso já é conhecido, mas sim avaliar se esse transbordo foi adequado ou não.

---

## 🎯 Objetivo da Entrega

A entrega busca classificar cada transbordo observado no fluxo EX3 → ONE em uma das seguintes categorias:

**Devido**, quando houver evidência de que o transbordo era esperado, necessário ou compatível com a natureza do caso.

**Indevido**, quando houver indícios de que o caso poderia ter sido tratado no EX3, mas foi encaminhado ao ONE sem necessidade real.

**Inconclusivo**, quando os dados disponíveis não permitirem afirmar com segurança se o transbordo foi devido ou indevido.

Além da classificação principal, a entrega poderá incorporar uma camada complementar de explicação, reunindo os principais sinais encontrados no processo para sustentar a classificação. Essa camada não pretende reproduzir toda a lógica de negócio em texto livre, mas sim tornar a análise mais interpretável e útil para revisões futuras.

---

## 🔎 Problema de Negócio que a Análise Resolve

Hoje o negócio percebe que existem transbordos entre sistemas, mas não possui uma leitura suficientemente estruturada para responder com segurança se esses transbordos representam:

* o comportamento normal do fluxo;
* limitações reais do EX3;
* exceções legítimas do processo;
* ou falhas operacionais e oportunidades de melhoria.

Sem essa distinção, o transbordo vira apenas um evento operacional registrado, mas não uma informação gerencial útil. Isso dificulta identificar gargalos, calibrar a atuação dos times, revisar escopo sistêmico e priorizar correções ou evoluções.

Ao classificar os transbordos e organizar as evidências observadas em cada caso, a análise passa a servir como instrumento de governança. Ela deixa de mostrar apenas “o que aconteceu” e passa a ajudar a responder “se deveria ter acontecido dessa forma” e “com base em quais sinais chegamos a essa conclusão”.

---

## 🧭 Princípio Analítico da Solução

A lógica da solução parte de um princípio simples: o EX3 informa que houve saída do caso, mas a melhor evidência sobre o motivo real do transbordo está no comportamento do caso dentro do ONE logo após essa transição.

Por isso, a análise não deve observar todo o histórico do processo no ONE. Isso traria ruído, porque um mesmo processo pode já ter tido movimentações anteriores naquele sistema e também continuará tendo novas movimentações depois que a etapa de subsídios terminar.

O foco da análise deve estar em uma **janela específica do processo**, delimitada por dois marcos:

o início da janela é o **momento do transbordo do EX3 para o ONE**;

o fim da janela é o **encerramento da etapa de subsídios**.

A população analisada será, portanto, o conjunto de eventos ocorridos no ONE entre esses dois marcos. Esse recorte é essencial para garantir que a leitura represente de fato a transição do subsídio e não seja contaminada por históricos anteriores ou por etapas posteriores do fluxo.

---

## ⏱️ Recorte da Análise

O recorte temporal da análise é uma das partes mais importantes da documentação, porque é ele que garante coerência entre o que o negócio quer responder e o que a pipeline efetivamente vai medir.

Serão considerados apenas os eventos do ONE que ocorram entre:

* o instante em que a pasta é transbordada do EX3 para o ONE;
* e o instante em que a etapa de subsídios é considerada encerrada.

Esse critério existe porque um mesmo processo pode possuir registros anteriores no ONE, sem relação com o transbordo que está sendo analisado, e também pode seguir gerando novos registros no ONE depois que a etapa de subsídios já terminou. Nenhum desses dois grupos de eventos interessa para esta análise.

Assim, a entrega não busca explicar toda a vida do processo no ONE, mas apenas a sua passagem pela etapa que pode justificar, ou não, o transbordo ocorrido a partir do EX3.

---

## 🧱 Como a Classificação Será Construída

A classificação será construída a partir da combinação de duas leituras.

A primeira leitura vem do **EX3**, que ajuda a identificar a natureza da saída do caso. Em termos de negócio, isso permite distinguir se o caso saiu do EX3 depois de ter sido tratado de forma compatível com o fluxo esperado ou se saiu porque houve alguma impossibilidade de continuidade naquele ambiente.

A segunda leitura, que é a principal, vem do **ONE**, por meio dos eventos registrados dentro da janela de análise. Esses eventos serão observados especialmente a partir de atributos como **tipos de solicitação** e **tipos de baixa**, pois são eles que representam o que efetivamente precisou ser tratado após o transbordo.

A ideia central é verificar se aquilo que foi tratado no ONE, naquele intervalo específico, era algo que o EX3 também teria capacidade de tratar ou não.

---

## 🗂️ Tabela de Elegibilidade

Para apoiar essa decisão, a solução prevê uma **tabela de elegibilidade**. Essa tabela funciona como um dicionário de negócio que responde, para cada combinação ou ocorrência relevante de evento, se aquele tratamento seria ou não compatível com o escopo do EX3.

Na prática, essa tabela não precisa registrar tudo o que existe no universo dos eventos. Ela pode ser construída de forma progressiva, priorizando os itens mais relevantes para o negócio e, especialmente, aquilo que já é conhecido como elegível ao EX3 ou como justificativa válida de exceção.

O mais importante é que essa tabela se torne a referência oficial da análise. Ela não substitui o conhecimento do negócio, mas o organiza de forma padronizada e rastreável para que a pipeline possa aplicar a mesma lógica de forma consistente em todos os casos.

Essa estrutura também favorece melhoria contínua, porque casos inicialmente classificados como inconclusivos podem ser revisitados conforme o mapeamento for amadurecendo.

---

## ⚖️ Definições de Classificação

### Devido

Um transbordo será considerado devido quando houver evidência de que ele era esperado ou necessário.

Isso pode acontecer em duas situações principais. A primeira é quando o caso foi tratado no EX3 conforme o fluxo previsto e depois seguiu normalmente para o ONE, sem que isso represente falha ou evasão de escopo. A segunda é quando, ao observar o comportamento do caso no ONE dentro da janela analisada, forem encontrados sinais de que o tratamento exigido ali não seria compatível com o escopo do EX3.

Em ambos os casos, a leitura é de que o transbordo teve fundamento operacional ou funcional legítimo.

### Indevido

Um transbordo será considerado indevido quando o caso tiver sido encaminhado ao ONE, mas os eventos observados dentro da janela de análise indicarem que o tratamento realizado ali poderia ter sido executado no EX3.

Em outras palavras, a leitura de indevido não significa necessariamente erro individual já comprovado, mas sim que a evidência disponível sugere que o transbordo era evitável à luz das regras conhecidas de elegibilidade.

Essa categoria é importante porque aponta oportunidades de correção operacional, revisão de entendimento de escopo e até de melhoria na utilização do EX3.

### Inconclusivo

Um transbordo será considerado inconclusivo quando os dados observados não permitirem concluir, com grau razoável de confiança, se o encaminhamento foi devido ou indevido.

Isso pode ocorrer, por exemplo, quando os eventos do ONE ainda não tiverem sido mapeados na tabela de elegibilidade, quando houver ambiguidades semânticas ou quando a evidência disponível for insuficiente para sustentar uma conclusão.

Essa categoria não deve ser lida como falha da entrega. Pelo contrário, ela é um sinal relevante sobre limites do dado, lacunas de mapeamento ou pontos do processo que ainda precisam ser melhor compreendidos.

---

## 🧾 Camada Complementar de Justificativa

Além da classificação final, a análise poderá trazer uma camada complementar de explicação. A proposta aqui não é construir textos complexos e artesanais para cada situação, porque isso tornaria a manutenção da lógica excessivamente difícil e pouco escalável.

A abordagem mais adequada é reunir, para cada pasta, os principais eventos encontrados na janela de análise que ajudam a sustentar a classificação. Esses eventos podem ser apresentados em formato de lista ou conjunto resumido, acompanhados de uma contagem simples da quantidade de sinais encontrados.

Essa camada enriquece muito a leitura porque permite que o negócio entenda não apenas o resultado final, mas também os indícios que levaram a ele. Com isso, a análise ganha poder explicativo, facilita revisões e ajuda a retroalimentar o próprio mapeamento de elegibilidade.

Na prática, isso significa que cada classificação pode vir acompanhada de algo como um conjunto de evidências observadas no ONE, apontando quais tipos de solicitação, tipos de baixa ou combinações similares sustentaram a decisão.

---

## 🔁 Valor Evolutivo da Entrega

Um dos pontos mais importantes desta solução é que ela não serve apenas para classificar casos individualmente. Ela também cria uma base para evolução contínua do entendimento do processo.

Ao acumular evidências sobre os casos classificados como devidos, indevidos e inconclusivos, a área passa a enxergar padrões. Isso pode ajudar a responder, por exemplo, quais situações aparecem com mais frequência como justificativa válida de transbordo, quais eventos se repetem nos casos potencialmente indevidos e quais itens ainda aparecem com recorrência sem mapeamento suficiente.

Com isso, a análise deixa de ser apenas um mecanismo de controle e passa a funcionar também como instrumento de aprendizado. Ela pode apoiar revisões de regra, melhorias operacionais, capacitação de times e até discussões sobre expansão ou ajuste do escopo funcional do EX3.

---

## 📦 Entregável Mínimo Esperado

O núcleo essencial da entrega consiste em disponibilizar, para cada pasta analisada, uma visão consolidada contendo pelo menos:

a identificação da pasta;

a informação de que houve transbordo do EX3 para o ONE;

a janela considerada na análise, isto é, o início no transbordo e o fim no encerramento da etapa de subsídios;

a classificação final do transbordo como devido, indevido ou inconclusivo.

Esse conjunto já é suficiente para entregar valor ao negócio, porque estabelece uma leitura objetiva e padronizada do mérito do transbordo.

---

## ✨ Entregável Enriquecido

Como camada adicional de valor, a entrega pode incluir também:

a quantidade de eventos observados dentro da janela de análise;

os principais tipos de solicitação e tipos de baixa encontrados nesse período;

um conjunto de evidências que sustentam a classificação;

a contagem dessas evidências;

e uma justificativa resumida, em linguagem simples, derivada da própria classificação e dos sinais encontrados.

Esse enriquecimento não é indispensável para que a análise exista, mas aumenta bastante sua utilidade prática, porque facilita exploração, revisão de casos e amadurecimento das regras de negócio.

---

## 👥 Público-alvo da Documentação

Esta documentação é voltada principalmente para pessoas de negócio, operação e gestão que precisam entender o objetivo da entrega, sua lógica de funcionamento e o valor que ela gera, sem depender de leitura técnica detalhada.

Por isso, o foco aqui não está em SQL, modelagem física ou detalhes de implementação. O foco está em explicar de forma clara:

* qual problema será resolvido;
* como a solução raciocina;
* quais decisões de negócio ela apoia;
* e quais limites e possibilidades devem ser considerados ao interpretar os resultados.

Uma documentação técnica complementar poderá ser produzida depois, com a tradução dessa lógica para regras de implementação, critérios de integração, definição dos campos, estrutura da pipeline e manutenção evolutiva.

---

## ✅ Resumo Executivo

Esta entrega tem como finalidade avaliar o mérito do transbordo entre EX3 e ONE na etapa de subsídios. Para isso, a análise utilizará a pasta como identificador transversal e observará apenas os eventos do ONE ocorridos entre o momento do transbordo e o encerramento da etapa de subsídios. A partir desse recorte, os eventos serão confrontados com uma tabela de elegibilidade que representa, de forma padronizada, o que seria ou não compatível com o escopo do EX3.

O resultado esperado é uma classificação de cada transbordo como devido, indevido ou inconclusivo, com possibilidade de enriquecimento por meio de evidências resumidas que expliquem a decisão. Com isso, a área de negócios ganha uma visão mais clara sobre a qualidade do fluxo entre sistemas, identifica oportunidades de melhoria e cria uma base consistente para evolução contínua das regras e do processo.

Se você quiser, no próximo passo eu posso transformar isso em uma versão já formatada como documentação formal, com seções como **Objetivo, Contexto, Escopo, Regras de Negócio, Entregáveis, Premissas, Limitações e Próximos Passos**.

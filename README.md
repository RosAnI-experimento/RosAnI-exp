# RosAnI 

Fluxo do agente de suporte helpdesk "RosAnI" com prompt otimizado para lidar com os manuais da empresa.

---

## Componentes: 
		 
-Chat input: Input do usuário (pode vir do playground ou do site) esse input é gravado na tabela message com a coluna Sender como “user” 

-OpenAi Embeddings: Usa o modelo de embedding configurado (nesse caso o text-3-small) para traduzir palavras em vetores, esse componente é acessado pelo componente do ChromaDB. Precisa da chave api da OpenAI pra funcionar. 
 
-ChromaDB: Banco de dados vetoriais onde os documentos estão armazenados, esse componente tem um campo para definir a coleção que será acessada (coleção é como um diretório, coleções diferentes podem possuir documentos diferentes), um campo para o diretório de persistência, que é a pasta onde os dados do chroma estão sendo gravados, input de search query que é um valor de input para uma 			 
pesquisa de similaridade vetorial no banco. Dentro dos controls desse componente tem um campo oculto que define o número de 			 
resustados que uma pesquisa retornar (number of results). 
 		 
-Parser: Esse componente formata a entrada de tipo Data ou DataFrame em uma saída de texto usando o tipo Message 
 		 
-Prompt: Este componente é feito para conectar variáveis definidas como {variável} para construir uma mensagem e enviá-la para uma LLM 	 
como System Message 
		 
-Message History: Este componente acessa o histórico de mensagens no banco (tabela message) mas também pode acessar de outro local 		usando o input de external memmory. Ele deve ser configurado para uso eficiente (parâmetros ocultos dentro da opção “controls”) 
 		 
-OpenAI: Componente de LLM da OpenAI onde podemos pegar um input e uma system message (prompt) e receber uma saída gerada pelo 		modelo. Nesse compoennte é possível escolher o modelo da OpenAI que será usado e a temperatura da resposta( temp maior significa 		halucinar mais). Possui alguns parâmetros internos (controls) de controle de uso de token porém é bem limitado. 
 
-Chat output: Componente padrão do LangFlow que pega uma entrada de tipo message e mostra ela no Playground (ou como resposta no 		site/API). Tudo que passar por esse componente é gravado na tabela message assim como o componente de chat input. 
 
OBS: 'Message' é um tipo de dado do LangFlow que contém informações de troca de conversa como role, content, name, metadata, etc. 

---

## Fluxo de dados: 
Começamos com a entrada do usuário no componennte 'chat input'. Essa entrada é enviada como parâmetro de pesquisa de similaridade (Para que essa pesquisa seja possível o fluxo primeiro chama o componente de embeddings para transformar a entrada do usuário em vetores e depois envia esses vetores para o ChromaDB).
O ChromaDB retorna em formato de DataFrame (tabela) os documentos mais parecidos com o conteúdo da pergunta, o parâmetro “number of results” do componente determinará quantos serão retornados.
Para que esses documentos possam ser lidos pelo agente, transformamos eles em uma única string concatenada usando o componente “Parser”, essa string é enviada para o componente prompt.
Para completar a base de conhecimento do agente e torná-lo capaz de entender o contexto de perguntas usamos o componente de Message History configurado para pegar as mensagem mais recentes.
O Componente de Prompt contém o prompt completo do agente e recebe o histórico de mensagens e os documentos em formato de string, todo esse conteúdo é passado para o componente da OpenAI como “System Message”.
O componente da OpenAI recebe a entrada inicial do usuário como input e, usando das instruções e documentos integrados no prompt gera uma resposta com base nos manuais, essa resposta é enviada para o componente de chat output como tipo message. 

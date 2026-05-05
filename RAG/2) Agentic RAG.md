~={yellow}Using Langraph , langchain , python=~
2 important concepts -> RAG + autonomous agent
![[Pasted image 20260504232355.png|562]]
In traditional RAG : only one path is followed. query taken to vector DB context is retrieved and the output is generated. 
Autonomous agent does : {Future is in developing such agents}
- When to retrieve
- what to retrieve
- where to retrieve
- how many times to retrieve

Workflow : ![[Pasted image 20260504233848.png|351]]

 ###  Using Langraph
 - Load open AI model
 `os.environ["OPENAI_API_KEY"]= os.getenv("OPENAI_API_KEY")  #get key from env`
`llm=ChatOpenAI(model="gpt-4.1" , temperature=0)
`embeddings=OpenAIEmbeddings()

- ~={yellow}Define state definiton =~: {How each state is defined}
`class AgentState(TypedDict):
`question : str
`documents : List[Document]
`answer : str
`needs_retrieval : bool
> Getting text to store in vector DB


`documents = [Document(page_content=text)  for text in sample_texts]
`vectorstore = FAISS.from_documents(documents , embeddings)
`retriever = vectorstore.as_retriever(k=2)

- ~={green}Agentic Function=~ {Define each node definition in the workflow}
![[Pasted image 20260504235142.png]]
`needs_retrieval : boolean if needed is turned to True
![[Pasted image 20260504235319.png]]
The context is stored in document variable mentioned in state definition.
![[Pasted image 20260504235429.png]]
![[Pasted image 20260504235457.png]]
get context and invoke prompt to llm

- Conditional Logic : Implement when there are two paths from a single node
Should go to which node
![[Pasted image 20260505000500.png]]
- Build the Graph
![[Pasted image 20260505001054.png]]
![[Pasted image 20260505001126.png]]
`app = worklfow.compile()
`app
 -  Testing 
![[Pasted image 20260505001346.png]]
`# Test with another question
`question2="How does RAG work?"
`result2 = ask_question(question2)
`print(f"Answer: {result2['answer']}")


## With MongoDB Vector Search
![[Pasted image 20260505142333.png|432]]

- Create cluster in MongoDB first
Get a connection url for creating MongoDB client
Use `pymongo to communicate with DB`

- Load your data embeddings model
`client=OpenAI()
`model="text-embedding-3-large"`

`def get_embedding(text, input_type="documents")
`response=client.embeddings.create(
`model=model,
`input=text
`)
`return response.data[0].embedding

- Data Ingestion to MongoDB
`loader=PyPDF("Path")
`data=loader.load()`
`text_splitter=RecursiveCharacterTextSplitter(chunk_size=400 , chunk_overlap=20)
`document=text_splitter.split_documents(data)`
 `doc_to_insert=[          ##Prepare documents for ingestion
 `{"text" : doc.page_content,
 `"embeddings" : get_embedding(doc.page_content)
 `}
 `for doc in documents 
 `]

`## Insert documents to mongoDB`
`from pymongo import MongoClient
`client=MongoClient("url")`
`collection=client.insert_many[][]
`result=collection.insert_many(doc_to_insert)

- Query with search Index
`from pymongo.operations import SearchIndexModel`
`index_name="Vector_index"`
`search_index_model=SearchIndexModel(
`definition = {
`"fields" : [
`{
`"type" : "vector",
`"numDimension" : 3072,
`"path" : "embedding",
`"similarity" : "cosine"
`}
`]
`}
`name=index_name,
`type="vectorSearch"
`)
`collection.create_search_index(model=search_index_model)

`query_embedding=get_embedding("query")
![[Pasted image 20260505163338.png]]
 Can also create , a `get_query_results function`


- Getting Response from LLM
`query=""
`context_docs=get_query_results(query)
`context_string="".join([doc["text"] for doc in context_docs])

`prompt=f""" Use the following pieces of context to answer the question
`{context_string}
`Question : {query}
`"""
`openai_client=OpenAI()
`model_name="gpt-4o"`
`completion=openai_client.chat.completions.create(
`model=model_name,
`messages=[
`{
`"role": "user",
`"content" : prompt
`}
`]
`)

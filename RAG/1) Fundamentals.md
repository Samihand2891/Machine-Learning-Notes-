RAG {Retrieval Augmented Generated} used for optimizing LLM knowledge base , by referencing to outside knowledge base of its training data .
We dont need to retrain the model 
![[Pasted image 20260501165612.png]]
Problem : 1) LLM is not updated upto to date so it will hallucinate on ques regarding new data
2) For updating your personal data on LLM you will need to fine tune it which is expensive as hell

Rag pipeline: Traditional Rag![[Pasted image 20260501170638.png]]
Query directly goes into the vector database. A vector database stores the data in the form of vectors. The query is converted into vectors and a similarity search is performed using Vector Database. Then we take the context and fill it with a prompt to the LLM and get the desired response. 
Data Ingestion Pipeline: data taken is parsed into k chunks and then converted into vectors using embedding. Stored in vector DB
Perplexity best example of RAG

- ~={green}Document Structure=~
Any file format in data ingestion i/p , 
![[Pasted image 20260501201130.png]]
Context size is fixed in Embeddings , so the given data ingestion parsed o/p should be converted into chunks. 

![[Pasted image 20260503003323.png]]
Page_content : All the data we want to read from file
metadata : Additional information about file in form of dictionary.
Loaders in Langchain : Load data from different file type (pdf ,csv ,excel)
Give output inform of document structure

WHY USE METADATA : Can be used to apply filters while chunking

ex: Text loader -
`from langchain.document_loaders import TextLoader`
`loader=TextLoader("path", encoding="utf-8")`
`document=loader.load() // loaded contents using text loader
O/P : {metadata : , contents: }
`dir_loader=DirectoryLoader(
`"path", glob="**/*.txt", ## Pattern to match files
`loader_cls= TextLoader,
`loader_kwargs= {specify encoding}
`)`

- Embedding and Vector store DB : Use sentence transformer , faiss-cpu/chromadb (for Vector DB)
`class EmbeddingManager :
`def __init__(self , model_name : str = "hugging face model"):
`self.model_name= model_name
`self.model=None
`self._load_model()`

- create a load model to load the model
`self.model=SentenceTransformer(self.model_name)

- Create a generate embeddings function
get text input and have numpy array output
`embeddings = self.model.encode(texts , show_progress_bar = True)`

VectorStoreDB
`class VectorStore:
`def __init__(self , collection_name : str = "name of collection", persist_directory: str = "/directory"):

`self.collection_name=collection_name
`self.persist_directory=persist_directory
`self.client=None
`self.collection=None
`self._initialize_store()`

- create a initalize_store function
self.collection for giving the place for where we store vectors

- add documents function 
To give the embeddings & documents as input
We zip (documents , embeddings ) and give a unique uuid(doc id variable)
`#Add to Collection
`self.collection.add( ids=ids, embeddings=embeddings_list, metadatas=metadatas, documents=documents_text)`
create a variable for each 

To add documents in collection of vector store
use `add_documents function
![[Pasted image 20260503172515.png]]
> Final text to embeddings to vectorDB:

- Creating A RAG Retrieval
`class RAGRetriever:
`def __init__(self , vector_store : VectorStore , embedding_manager: EmbeddingManager):
`self.vector_store= vector_store`
`self.embedding_manager= embedding_manager`

- create a retrieve function : 
`def retrieve(self , query : str , top_k: init =5 , score_threshold : float = 0.0) -> List[Dict]
Retrieves relevant documents for a query
returns List of dictionaries containing retrieved documents and metadata

While getting query it needs to be converted into embeddings too ,
`query_embedding= self.embedding_manager.generate_embeddings([query])`
- We search the vector DB according to the query
- Get top_k results {documents , metadats , distances , ids}
- Calculate `similarity_score=1-distance`
- Add to context documents 

Call a query :
`rag_retreiver=RAGRetirever(vectorstore , embedding_manager)
`rag_retriever.retrieve("What is attention is all you need)


- Retrieval Pipeline {User context -> Give to LLM
![[Pasted image 20260503183125.png]]
Context + prompt = Augmentation 
O/P : Generation
1) Initialize groq LLM in your environment
2) Create a RAG function : retrieve context + generate response
`def rag_simple(query , retriever , llm , top_k=3)`
![[Pasted image 20260503184257.png]]
`answer=rag_simple("QUestion", rag_retriever , llm)`
`print(answer)`

- Ehnanced Pipeline : {sources , confidence score & optionally full context}


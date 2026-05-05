![[Pasted image 20260505183144.png]]
Use Langsmith 
### LLM Chatbot 
- ~={purple}Chatbot evaluation=~
1) Signup to Langsmith and get a API keyf
`load_dotenv()
`os.environ["LANGSMITH_API_KEY"]=os.get_env("LANGSMITH_API_KEY")
`os.environ["OPEN_AI_KEY"]=os.getenv("OPEN_AI_KEY")

- Create the datapoints
Use Datasets & Experiments module in Langsmith to create datasets and also evaluate it by performing experiments.

`client=Client()  define the client

`## Define test dataset`
`dataset_name=""`
`dataset=client.create_dataset(dataset_name)
`client.create_examples(
`dataset_id=dataset.id,
`examples=[]
`)`
The examples are list of dictionaries with i/p question and output answer

- ~={orange}Create a LLM as Judge ( Defining metrics)=~
`openai_client=wrappers.wrap_openai(openai.OpenAI())
Used to see what calls did the LLM make
`eval_instructions= "Expert in evaluating students answers to questions"`

`def correctness(inputs : dict , outputs : dict , reference_outputs: dict) -> bool:`
`user_content=f""" """
`response= openai_client.chat.completion.create(
`model="gpt-4o-mini",
`temperature=0,
`messages=[
`{"role" : "system", "content": eval_instructions},
`{"role" : "user", "content" : user_content}
`]

`).choices[0].message.content

`return response ==  "CORRECT"

`# Another metric Concision : checks whether the actual output is less than 2 times the length of the expected result 
`def concision(outputs:dict , reference_outputs : dict) -> bool:
`return int(len(outputs['response'])) < 2 * len(reference_outputs["answer"])

- Run Evaluations for Chatbot
` default_instruction= "Respond to the users question in a short , concise manner(one short sentence)"`
`def my_app(questions : str , model : str = "gpt-4o-mini", instructions : str = default_instructions)
`return openai_client.chat.completion.create(
`model="gpt-4o-mini",
`temperature =0,
`messages =[
`{"role" : "system", "context": instructions},
`{"role" : "user" , "context" : "question"},
`],
`).choices[0].message.content

`# Call my_app func for every datapoints`
 `def ls_target(inputs: str) -> dict:
	 `return{"responses" : my_app(inputs["question"], model="")} # can also give different model 

`# Run Evaluation `
`exp_results=client.evaluate(
`ls_target , # Your AI system
`dataset=dataset_name,
 `evaluators=[correctness , concision]
`experiment_prefix = "gpt-4o-mini-chatbot"
)`

Go to langsmith to see results
![[Pasted image 20260505205727.png]]



### RAG EVALUATION
1) How to create test datasets?
2) RUN Rag APP -> Test datasets
3) Measure RAG performance -> Different evaluation metrics
![[Pasted image 20260505211214.png]]
We should evaluate if documents are really relevant?
Is answer grounded in document?
Answer correctness also measured , also does answer addresses the relevant question.

Steps :
1)  RAG -> DATA ingestion , retriever , generator
2) TEST DATA  => question / answer
3) Evaluation metrics {based on upper points} [Use llm as judge]

- First define a chatmodel , 
`llm= init_chat_model("openai:gpt-4o-mini")`

`@traceable()
`def rag_bot(question : str) -> dict :
	`docs=retriever.invoke(question)
		`docs_string ="".join(doc.page_content for doc in docs) 
- Create a prompt and invoke llm



- Create  a DATASET
`client=Client()`
`examples=[] list of dictionaries

`# Create dataset and example in langsmith`
`dataset_name=""
`dataset=client.create_dataset(dataset_name=dataset_name)
`client.create_examples(
`dataset_id=dataset.id,
`examples=examples
`)


-   Evaluation RAG
1) Correctness : Response vs reference answer
- Goal : Measure "how similar / correct is the rag chain answer relative to a ground answer"
- Mode : Requires a ground truth reference answer is supplied through a data set 
- Evaluator : Use LLMS, Judge to assess answer correctness. 

`correctness_instructions =""

2) Relevance : Response v/s Input : The flow is similar to above , but we simply look at the inputs and outputs without needing the reference_outputs. Without a reference answer we can't grade accuracy , but can still grade relevance- as in , did the model address the user's question or not

3) Groundedness : Response v/s retrieved docs  : Another useful way to evaluate responses without needing reference answers is to check if the response is justified by (or "grounded in") the retrieved documents.

4) Retrieved relevance : Input v/s retrieved docs

- Run Evaluation
![[Pasted image 20260505235613.png]]

> Note : Can add different metrics , also model for better evaluation scores
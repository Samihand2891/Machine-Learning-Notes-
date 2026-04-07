## Course contents:
1) Building a API : Social media 


==How to setup a env :==
**command** : `py -3 -m venv name`
1) setup new interpreter from View - Interpreter and provide it path **./Scripts/python.exe**
2)  Also provide terminal activate.bat to activate venv in terminal
**Documentation** : https://fastapi.tiangolo.com/tutorial/#install-fastapi

##### Main .py
`from fastapi import FastAPI`
`app =FastAPI()

`@app.get("/") - decorator ("/") tells the path we have to go through url
`def root():
`return {"message": "Hello World"}` 

Path operations : the def function can contain anything you want to perform
How to connect to Http- Use `uvicorn main:app`
%% Fast Api converts text to JSON %%
 
 IMP : To change anything in server we have to restart or use `uvicorn main:app --reload`
 ==Understand http requests== : https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
  Fast api  `.get("")` goes to first match of path and stops running
  
**POSTMAN** : To test api by sending data to server 

Difference between POST and GET request :
1) In POST request you can also send data to api server
![[Pasted image 20260407091211.png|526]]
How to send data to POST : 
`@app.post("/create_post")     USe Postman - Body -JSON on given url
`def create_posts() :`
`return {"msg": "Post created"}`
~={yellow}Then, =~
edit post request to 
`def create_posts(payload : dict=Body(...)) :`
`print(payload)`

##### Schematic Validation with Pydantic
> Need to define a schema of what data should look like for input which the client inputs    

Docs : https://docs.pydantic.dev/latest/api/config/
`class Post(BaseModel) :
`title : str      //type of data 
`content : str 
`published bool = True // Default true unless mentioned by user
`@app.post("/")
`def create_posts(new_post :Post) : // Referencing to class and checks if input is                                            in same schema as class
`print(new_post)

To send back data in form of dictionary,
`print(new_post.dict())
`return {"data" : "new_post"}

#### Crud application
![[Pasted image 20260407101048.png|559]]

****Create:**** It involves adding new data to the database. In FastAPI, we can create data by sending a POST request to the appropriate endpoint. For example, to add a new entry to the database, we would send a POST request to the creation endpoint with the relevant details in the request body.

****Read:**** It involves retrieving existing data from the database. In FastAPI, we can perform Read operations using GET requests. For example, if we want to retrieve any entry from a database, we would send a GET request to the respective endpoint.

****Update:**** It involves updating existing data in the database. In FastAPI we can perform an Update operation using PUT/PATCH, PUT allows us to update the entire database whereas PATCH allows us to modify specific fields in the database. For example, if we want to update any attribute of the user we would send a PUT/PATCH request to the endpoint with new details.

****Delete:**** It involves deleting existing data from the database. In FastAPI we can perform a Delete Operation using DELETE requests. For example, if we want to delete any entry of a user we would send a DELETE request to the endpoint.

Storing data in memory , 
`my_posts=[{"title" : "title 1" , "contents" : "contents of 1" , "id" : 1 }]
in database each post has a unique id 
Update : `@app.get("/post:)
`return {"data" : my_post} to gain data from get request and store in memory

Through post operation,
Update: `@app.post("/post")
`my_post.append(post.dict())

Get a single individual post ,
`@app.get("/post/{id}")
`def get_post(id :int ) : // the id should be an integer so we define the dtype

`Fastapi` accesses the id directly provided by user so we can use the id to get the individual data

Http Status codes , 
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
Error 404 : not found
`def get_post(int : id , response=Response): Response can be altered however you want
`if not in post :
`response.status_code=404 / status.{HTTP code}
or raising HTTP exception,
`if not in post :
`raise HTTPException(status_code=status.HTTP_error_404_not_found , detail="message" )
>  for post operations

`@app.post("/post", status_code=HTTP_code)

Delete ,
`@app.delete("/post/{id}")  
`def delete_post():
`index=find_index_post(id)
`my_posts.pop(index)      
`return Response(status_code=status.HTTP_204_NO_CONTENT)` important to not send data back in 204 error
![[Pasted image 20260407165738.png]]
  for finding index wrt to id 

Update, 
`@app.put("/posts/{id})` In put operation have to give all fields value in update req
`def update_post(id : int , post: Post) : // for post: Post defining the schema 
`index = find_index_post(id)
`if index == None :
`raise HTTPexception (status_code=status.HTTP_CODE_NOT_FOUND`
`post_dict=post.dict()
`post_dict["id"]=id
`my_posts[index]=post_dict

Build in Documentation, 
Go to link : http://127.0.0.1:8000/docs
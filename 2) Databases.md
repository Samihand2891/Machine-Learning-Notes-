~={yellow}Creating a Python package :=~


 - Always in a folder intialized a empty folder named ____init____.py
 - to reference uvircorn : uvicorn app.main: app --reload


Using Postgres
-- 
Databases : Table used to represent subject or event in an application
In a relational database all tables are related.
Need to specify column __datatype__ :https://www.postgresql.org/docs/current/datatype.html
![[Pasted image 20260408221033.png|566]]
Primary Key : Can be any column , a primary key is basically a unique identifier ,
Constraints - Unique , Null

Postgres server ,
connect the url to the instance of whatever you want to join

> SQl code to open postgres server thorugh terminal :
`CREATE DATABASE fastapi
    `WITH
    `OWNER = postgres
    `ENCODING = 'UTF8'
    `LOCALE_PROVIDER = 'libc'
    `CONNECTION LIMIT = -1
    `IS_TEMPLATE = False;
- Server - Databases-schema-tables (for creating tables)
- Timestamp Datatype: time with time zone and Default : Now()

##### Query tool : 
`SELECT * FROM tablename; //Prints all table value , * means all values
`SELECT columnname from tablename;
> capitalization doesnt matter but using sql specific commands in capital is good practice

`SELECT id as product_id FROM tablename; 
only id column will be shown but with name product_id
`SELECT * FROM tablename WHERE id =10;
Get all column values of id =10
For text , `WHERE name='TV' , can use > , < ,= , AND , OR , !

Use regex for char dtype ,
`SELECT * FROM tablename WHERE columnname LIKE 'yourregex' ;
Use order by,
`SELECT* FROM tablename ORDER BY columnname ASC/DESC;
`SELECT * FROM tablename ORDER BY inventory DESC , price ASC;
Setting limit to output , 
`SELECT * FROM tablename LIMIT 10;
`SELECT * FROM tablename LIMIT 10 OFFSET 2; // skips first 2 and takes next 10
Command to add data to DB :
`INSERT INTO tablename (name , price , inventory) VALUES ('tortilla' , 4 , 2000);
                      columns                                                     

Using return at end :
`returning * // returns the newly added row
Multiples rows to be added ,
`INSERT INTO tablename (name , price , inventory) VALUES ('tortilla' , 4 , 2000), ('apple',4,300);
Deleting rows ,
`DELETE FROM tablename WHERE id=10;
Get column back which was deleted,
`DELETE FROM tablename WHERE id=10 RETURNING *;

Update Database,
`UPDATE tablename SET name='flour tortilla', price=40 WHERE id =25;
                (what values you want to change)
 








#### Psycopg 
Connecting to database,
`conn.connect(host='localhost', database='fastapi', username ='', password='', cursor_factory=RealDictCursor)
`cursor=conn.cursor() using to execute sql command

Retrieving data using get operation ,
In , `@app.get('/posts')
`def get_posts():
`cursor.execute(""" SELECT * FROM ;""")
`posts=cursor.fetchall()` // fetching data from sql data
`print(posts)`

Updating data in post operation,
`@app.post()
`def create_post(post: POST):`
`cursor.execute("""INSERT INTO tablename (), VALUES (%s, %s, %s) , RETURNING *""", (post.title , post.content , post.published))
`new_post=cursor.fetchone()
`conn.commit() Important command needed to push the changes to databases
- Create a new variable to get the returning statement



###  Dealing with ORMS(SQLALCHEMY)

> [!NOTE]
> Object relational mapper : A layer of abstraction that sits b/w database and us

- Can perform traditional database operations without sql
![[Pasted image 20260411212432.png|443]]
- Instead of manually defining tables we can define tables as python modules

Using SQL alchemy, 
- For using SQL alchemy you need to install the database driver you are using
https://fastapi.tiangolo.com/tutorial/sql-databases/
- Every database has a unique url which connects to sql alchemy for postgres:
`SQL_ALCHEMY_DATABASE='postgresql://<username>:<password>@<ip-address/hostname>/<database_name>'
`engine = create_engine(SQL_ALCHEMY_DATABASE)
`Sessionlocal=  sessionmaker(autocommit=False, autoflush=False, bind=engine)
`Base = declarative_base()`
From docs

Defining models, 
`class Post(Base) :
`__tablename__="posts"
`id=Column(Integer , primary_key=True, nullable=False)
`title=Column(String ,nullable=True )
`content=Column(String , nullable=True)
`published=Column(Boolean, server_default='TRUE', nullable=False)
`created_at=Column(String, server_default=text('now()'), nullable=False)

Session dependency ,
every API call you have to define session wrt to get_db()
`@app.get("/sqlalchemy")
`def test_posts(db: Session=Depends(get_db)) :

`def get_db():
`db=Sessionlocal()
`try:
      `yield db
`finally:
       `db.close()

- SQL ALCHEMY does not do data migration 

~={orange}Performing sql operation:=~
using : db.query()
- Returning data =>  `posts=db.query(models.Post).all()
`return {"data"=posts}
- Creating data =>
`def create_posts(post:Post , db : Session = Depends(get_db)):
`new_post=models.Post(title=post.title , content=post.content , published=post.published) // used to fill in what new data we want
`db.add(new_post) //Adding the post
`db.commit() // Commiting changes
`db.refresh(new_post)`// Getting the data back

Easier way to do this ,
`new_post=models.Post(** post.dict()) //Unpacks the input dict into perfect format according to schema

- Getting one individual data =>
`@app.get('/posts/{id}')
`def get_post(id : int , db : Session =Depends(get_db))`
`post_w_id=db.query(model.Post).filter(model.Post.id==id).all().first()`
// the first operator is used so that postgres does not waste time looking over all data to find the data with exact id again

- Deleting data =>
 `app.delete('/posts/{id}', status_code=status.HTTP_204_NO_CONTENT)`
 `def delete_posts(id : int , db: Session = Depends(get_db)) :
 `post_del=db.query(model.Post).filter(model.Post.id==id).all()
`if post_del.first() == None :
`     raise HTTP_Exception (status_code= status.HTTP_404_NOT_FOUND)
`post.delete(synchronize_session=False
`db.commit()
`db.refresh(post_del)

- Update data =>
`def update_post(id:int , db: Session =Depends(get_db))
`post_update= db.query(model.Post).filter(model.Post.id==id)`
`if post_update.first() == None
`       raise HTTP_Exception (status_code=status.HTTP_404_NOT_FOUND)
`post_update.update(post.dict(), synchronize_session=False)`
`db.commit()

Difference b/w Pydantic & ORM

![[Pasted image 20260412222521.png|456]]

Use pydantic schema : 
`class PostBase(Basemodel) :
`title: str
`content: str
`published: bool =True`

`class CreatePost(PostBase) :
`Your command

`class UpdatePost(PostBase)

Not Sending unwanted things back , '
Define response class : 
`class Response(BaseModel) :
`title : str
`content : str
`published : bool= True
     `class Config:
         orm_mode=True

in Decorator -- `@app.post("/posts", status_code=status.HTTP_201_CREATED, response_model=schema.Response)

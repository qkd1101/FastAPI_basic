# Fast api Document

## 1.What is FastAPI
>FastAPI is a high performant web framework.   

## 2.FastAPI setting
### 2.1 FastAPI basic
**2.1.1 Install FasAPI**   
`$pip install fastapi`   or
`$pip3 install fastapi`//this is for python version3
   
**2.1.2.For Fast Api production**   
`$pip install unicorn`	//python version2   
`$pip install hypercorn`   
   
*If you want to change python default version   
`$ln -s -f /usr/local/bin/python3.9 /usr/local/bin/python`   
And exit terminal, reopen it.   
   
Check version :   
```shell
python --version
```  
`Python 3.9.5`   
   
**2.1.3Check fastapi**   
   
Create main.py   
``` python
from fastapi import FastAPI   
  app = FastAPI()   

  @app.get("/")   
 def read_root():    
    return {"Hello": "World"}
```
Terminal   
```shell
uvicorn main:app --reload 
``` 
 check 127.0.0.1:8000   
   
   
### 2.2FastAPI For Docker
**2.2.1Create API DIR**   
`$mkdir project-name`   

**2.2.2Create Dockerfile**    
```shell
touch Dockerfile 
```
Dockerfile
```
FROM python:3.7   

RUN pip install fastapi uvicorn   

EXPOSE 80   

COPY ./app /app   

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]   
```

**2.2.3Create docker-compse.yml**   
   
docker-compse.yml
```yml
version: '3'   
services:    
    core-api:    
        build: .    
        container_name: //container name   
        ports:    
            - "8000:8080"   
        volumes:    
            - ./app:/app   
```

**2.2.4Follow command terminal**   
```shell
docker-compose up -d  
```

**2.2.5Check container and run**   
```sehll
docker images   
docker ps
```
*https://www.youtube.com/watch?v=gVymPpepQco   
   
```shell
docker main:app --reload
```
   
and check 127.0.0.1:[yourport]/docs   
this is way to DocsUI
## FastApi schema
>FastAPI create schema 3 ways

### 1.
```python
...
from pydantic import BaseModel
...

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

    class Config:
        schema_extra = {
            "example": {
                "name": "Foo",
                "description": "A very nice Item",
                "price": 35.4,
                "tax": 3.2,
            }
        }

```

### 2
```python
from pydantic import BaseModel, Field
...
class Item(BaseModel):
    name: str = Field(..., example="Foo")
    description: Optional[str] = Field(None, example="A very nice Item")
    price: float = Field(..., example=35.4)
    tax: Optional[float] = Field(None, example=3.2)

```

### 3
```python
...
from fastapi import Body, FastAPI
from pydantic import BaseModel
...
class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Item = Body(
        ...,
        examples={
            "normal": {
                "summary": "A normal example",
                "description": "A **normal** item works correctly.",
                "value": {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                },
            },
            "converted": {
                "summary": "An example with converted data",
                "description": "FastAPI can convert price `strings` to actual `numbers` automatically",
                "value": {
                    "name": "Bar",
                    "price": "35.4",
                },
            },
            "invalid": {
                "summary": "Invalid data is rejected with an error",
                "value": {
                    "name": "Baz",
                    "price": "thirty five point four",
                },
            },
        },
    ),
):
    results = {"item_id": item_id, "item": item}
    return results
```
And check 127.0.0.1:[your port]/docs or   
127.0.0.1:[your port]/redoc

>you can get example code 3 at docs/app/json_object_schema. and the other examples are https://fastapi.tiangolo.com/tutorial/schema-extra-example/.   

## POST JSON object
code
```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```
and use `curl` command

```shell
curl -X 'POST' \
  'http://127.0.0.1:8000/items/' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "foo",   
  "description": "foo",   
  "price": 10,
  "tax": 20
}'

```

then you can get

```shell
{"name":"foo","description":"foo","price":10.0,"tax":20.0,"price_with_tax":30.0}
```

and check 127.0.0.1/[your port]/docs.   

you can try this here.

## POST JSON Object with java code

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;
 
public class mainClass {

	public static void main(String[] args) throws Exception {
		sendPostJson();
	}

	public static void sendPostJson() throws Exception{
		URL url = new URL("http://127.0.0.1:8000/items/");
		
		HttpURLConnection con = (HttpURLConnection)url.openConnection();
		con.setRequestMethod("POST");
		con.setRequestProperty("Content-Type", "application/json");
		con.setRequestProperty("Accept", "application/json");
		con.setDoOutput(true);
		
		String jsonInputString = "{\"name\": \"foo\",  \"description\": \"foo\", \"price\": 30,  \"tax\": 40}";
		
		try(OutputStream os = con.getOutputStream()){
			byte[] input = jsonInputString.getBytes("utf-8");
			os.write(input,0,input.length);
		}
		
		try(BufferedReader br = new BufferedReader(
				new InputStreamReader(con.getInputStream(),"utf-8"))){
			StringBuilder response = new StringBuilder();
			String responseLine = null;
			while((responseLine = br.readLine())!=null) {
				response.append(responseLine.trim());
			}
			System.out.println(response.toString());
		}
		
	}
}
```

then you can check at console

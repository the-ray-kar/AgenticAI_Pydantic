## Limitations of Python and How Pydantic Solves Them

### 1. **Lack of Built-in Data Validation**
- Python's built-in `dataclasses` and regular class attributes do not enforce type validation at runtime.
- Example:
  ```python
  from dataclasses import dataclass

  @dataclass
  class User:
      name: str
      age: int

  user = User(name="Alice", age="twenty")  # No error, even though age is a string
  ```

#### **Pydantic Solution**
- Pydantic ensures type validation at runtime:
  ```python
  from pydantic import BaseModel

  class User(BaseModel):
      name: str
      age: int

  user = User(name="Alice", age="twenty")  # Raises ValidationError
  ```

### 2. **Manual Data Parsing and Transformation**
- Python does not automatically convert values (e.g., string to integer) when assigning attributes.

#### **Pydantic Solution**
- Pydantic automatically parses and converts data:
  ```python
  class User(BaseModel):
      age: int

  user = User(age="25")  # Converts "25" to an integer
  ```

### 3. **Nested Models Handling**
- Python lacks a structured way to validate nested models.

#### **Pydantic Solution**
- Pydantic supports nested models effortlessly:
  ```python
  class Address(BaseModel):
      city: str
      zip_code: int

  class User(BaseModel):
      name: str
      address: Address
  
  user = User(name="Alice", address={"city": "NY", "zip_code": "10001"})
  ```
  - Pydantic will convert `zip_code` to an integer automatically.

### 4. **Complex Data Validation**
- Python requires custom logic to validate fields beyond type hints.

#### **Pydantic Solution**
- Pydantic allows defining custom validation rules:
  ```python
  from pydantic import validator

  class User(BaseModel):
      age: int

      @validator("age")
      def check_age(cls, value):
          if value < 18:
              raise ValueError("Age must be 18 or older")
          return value
  ```

### 5. **JSON Schema Generation**
- Python does not provide built-in tools for generating JSON schemas for data models.

#### **Pydantic Solution**
- Pydantic can generate JSON schemas automatically:
  ```python
  print(User.schema_json())
  ```

---

## Additional Features of Pydantic

### 6. **Field Customization and Aliases**
- Pydantic allows renaming fields for serialization/deserialization:
  ```python
  from pydantic import Field

  class User(BaseModel):
      full_name: str = Field(..., alias="fullName")
  
  user = User(fullName="Alice Doe")
  print(user.dict(by_alias=True))  # {"fullName": "Alice Doe"}
  ```

### 7. **Environment Variable Parsing**
- Pydantic can load settings from environment variables:
  ```python
  from pydantic import BaseSettings

  class Settings(BaseSettings):
      database_url: str
  
  settings = Settings()
  print(settings.database_url)  # Reads from environment variables
  ```

### 8. **Custom Data Types**
- Pydantic supports custom types and constrained fields:
  ```python
  from pydantic import conint

  class User(BaseModel):
      age: conint(gt=18)  # Age must be greater than 18
  ```

### 9. **Asynchronous Validation**
- Pydantic supports async validation with `@validator`:
  ```python
  from pydantic import BaseModel, validator
  import asyncio

  class User(BaseModel):
      name: str

      @validator("name")
      async def validate_name(cls, value):
          await asyncio.sleep(1)  # Simulate async operation
          return value.title()
  ```

### 10. **ORM Mode for Integration with Databases**
- Pydantic models can work with ORMs like SQLAlchemy:
  ```python
  class User(BaseModel):
      id: int
      name: str
      class Config:
          orm_mode = True
  ```

### 11. **Strict Mode for Type Checking**
- Pydantic allows strict mode to enforce exact types:
  ```python
  class User(BaseModel):
      age: int

      class Config:
          strict = True
  ```
  - This will prevent automatic conversions (e.g., string to int).

### 12. **Exporting and Serialization**
- Pydantic models can be exported to various formats:
  ```python
  user = User(name="Alice", age=25)
  print(user.json())  # JSON format
  print(user.dict())  # Dictionary format
  ```

### 13. **Reading and Parsing JSON Data**
- Pydantic can directly parse JSON data into models:
  ```python
  import json
  
  json_data = '{"name": "Alice", "age": 25}'
  user = User.parse_raw(json_data)
  print(user)
  ```
- It also supports loading from files:
  ```python
  with open("user.json", "r") as f:
      user = User.parse_raw(f.read())
  ```
- Another method is using `parse_obj` for dictionaries:
  ```python
  data = {"name": "Alice", "age": 25}
  user = User.parse_obj(data)
  ```

### Conclusion
Pydantic enhances Python by enforcing runtime type validation, automatic data parsing, nested model handling, custom validation, and JSON schema generation. Additionally, it provides field customization, async validation, ORM support, strict mode, environment variable parsing, and efficient JSON reading and parsing, making it a powerful tool for data handling in Python applications.

## Understanding `Field` in Pydantic

In Pydantic, the `Field` function is used to add extra metadata, constraints, and default values to model attributes. It enhances validation beyond simple type hints.

### **Usage of `Field`**
The `Field` function is imported from `pydantic` and allows defining constraints such as minimum/maximum values, regex patterns, default values, and descriptions.





```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., min_length=3, max_length=50, description="User's full name")
    age: int = Field(..., gt=0, lt=120, description="User's age (must be between 1 and 119)")
    email: str = Field(..., regex=r"^[\w\.-]+@[\w\.-]+\.\w+$", description="Valid email address")

user = User(name="Alice", age=30, email="alice@example.com")
print(user)
```

### **Key Features of `Field`**

#### 1. **Set Default Values**
```python
class User(BaseModel):
    country: str = Field("USA")  # Default value
```

#### 2. **Set Constraints**
```python
class Product(BaseModel):
    price: float = Field(..., gt=0, lt=10000)  # Price must be > 0 and < 10000
```

#### 3. **Add Metadata (Descriptions & Titles)**
```python
class User(BaseModel):
    bio: str = Field(..., description="User's biography", title="Biography")
```

#### 4. **Use Regular Expressions for Validation**
```python
class Contact(BaseModel):
    phone: str = Field(..., regex=r"^\+?\d{10,15}$")  # Validates phone numbers
```

#### 5. **Mark Optional Fields with `None`**
```python
from typing import Optional

class User(BaseModel):
    nickname: Optional[str] = Field(None, description="User's nickname (optional)")
```

### **When to Use `Field`?**
- When you need validation beyond simple type hints.
- When you want to document your model with descriptions.
- When providing default values or constraints.

Using `Field` in Pydantic makes data validation more powerful and structured, improving data integrity in applications.

### Simple Agent without tool but output in Specified format

Ensuring the GEMINI_API_KEY is in env. You can create a simple agent which return output
```python
import asyncio
from pydantic import BaseModel, Field
from pydantic_ai import Agent

# 1. Define output model
class ChoiceOutput(BaseModel):
    index: int = Field(..., description="Index of the selected option (0-based)")

# 2. Initialize the agent with Gemini
agent = Agent(model="google-gla:gemini-2.0-flash",output_type=ChoiceOutput)

# 3. Async runner function
async def run_agent():
    choices = ["cat", "dog", "bird"]
    prompt = f"""
    You are a helpful assistant.

    Here is a list of choices:
    {', '.join(choices)}

    Please pick the most appropriate one and respond in JSON like this:
    {{"index": <index>}}

    ONLY return the JSON, nothing else.
    """

    result = await agent.run(user_prompt=prompt)
    return result

# 4. Run the agent
result = await run_agent()
print(result.output.index)  # Output will be an integr

```


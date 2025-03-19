# **Backend API Test Automation with FastAPI and Requests**

In this tutorial, we'll cover how to automate backend API testing using **FastAPI** for the server and **Requests** for testing. We'll follow a structured approach:

1. **Setting Up the FastAPI Server**
2. **Running the Server**
3. **Writing Automated Tests using Requests**
4. **Enhancing Tests with Pytest**
5. **Expanding the Idea for Real-World Projects**

---

## **1. Setting Up the FastAPI Server**
We'll first create a simple API with three endpoints: **add, subtract, and multiply**.

### **Install FastAPI and Uvicorn**
If you haven't already, install FastAPI and Uvicorn:

```sh
pip install fastapi uvicorn
```

### **Create the FastAPI Server (`api.py`)**
```python
from fastapi import FastAPI, Query
from pydantic import BaseModel

app = FastAPI()

class Result(BaseModel):
    result: int

@app.get("/", response_model=dict)
def read_root():
    return {"message": "Welcome to the FastAPI Math API"}

@app.get("/add", response_model=Result)
def add(num1: int = Query(..., description="First number"), num2: int = Query(..., description="Second number")):
    return {"result": num1 + num2}

@app.get("/subtract", response_model=Result)
def subtract(num1: int = Query(...), num2: int = Query(...)):
    return {"result": num1 - num2}

@app.get("/multiply", response_model=Result)
def multiply(num1: int = Query(...), num2: int = Query(...)):
    return {"result": num1 * num2}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run("api:app", host="0.0.0.0", port=8000, reload=True)
```

### **Key Improvements**
**Query Parameters instead of Path Parameters**  
Instead of `/add/2/3`, you can now use `/add?num1=2&num2=3`, making it more user-friendly.

**Pydantic for Type Validation**  
We define a `Result` model to ensure correct API responses.

**Default Root Endpoint (`/`)**  
A simple welcome message when accessing the base URL.

---

## **2. Running the Server**
Start the FastAPI server:

```sh
python api.py
```

Your API will be available at:

```
http://localhost:8000
```

You can test it using:

```
http://localhost:8000/add?num1=2&num2=3
```

FastAPI also provides automatic documentation:

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

---

## **3. Writing Automated API Tests**
Now, let's automate API testing using the `requests` library.

### **Install Requests**
```sh
pip install requests
```

### **Basic Test Script (`test_api.py`)**
```python
import requests

BASE_URL = "http://localhost:8000"

testcases = [
    ("/add?num1=2&num2=2", 4),
    ("/subtract?num1=2&num2=2", 0),
    ("/multiply?num1=2&num2=2", 4),
]

def test():
    for endpoint, expected in testcases:
        response = requests.get(BASE_URL + endpoint)
        assert response.json()["result"] == expected
    print("All tests passed!")

if __name__ == "__main__":
    test()
```

### **Run the Tests**
```sh
python test_api.py
```

**If all tests pass**, you’ll see:
```
All tests passed!
```

**If a test fails**, it will throw an `AssertionError`.

---

## **4. Enhancing Tests with Pytest**
Instead of a simple script, we can use `pytest` for better testing.

### **Install Pytest**
```sh
pip install pytest
```

### **Refactored Test Script with Pytest (`test_api.py`)**
```python
import pytest
import requests

BASE_URL = "http://localhost:8000"

testcases = [
    ("/add?num1=2&num2=2", 4),
    ("/subtract?num1=2&num2=2", 0),
    ("/multiply?num1=2&num2=2", 4),
    ("/add?num1=-1&num2=5", 4),   # Test negative numbers
    ("/multiply?num1=0&num2=5", 0) # Test zero multiplication
]

@pytest.mark.parametrize("endpoint, expected", testcases)
def test_math_operations(endpoint, expected):
    response = requests.get(BASE_URL + endpoint)
    assert response.status_code == 200, f"Failed: {endpoint}"
    assert response.json()["result"] == expected
```

### **Run Tests with Pytest**
```sh
pytest test_api.py
```

**Advantages of Using Pytest**
- Better test organization  
- Detailed error reporting  
- Can run multiple test files easily

---

## **5. Expanding the Idea for Real-World Projects**
Now that we have a basic API with automated tests, how can this be applied in real-world projects?

### **1️⃣Add Database Integration**
Instead of simple arithmetic, imagine an API that fetches/stores data in a **PostgreSQL** or **MongoDB** database. Testing would involve:
- **Checking data integrity** after each API call.
- **Mocking database connections** for isolated testing.

### **2️⃣Authentication and Authorization**
- Add **OAuth2, JWT, or API keys** for secure endpoints.
- Test unauthorized access to ensure security.

### **3️⃣CI/CD Integration**
- Use **GitHub Actions / Jenkins** to automate testing on each code push.
- Example **GitHub Action for Pytest**:

```yaml
name: API Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: "3.10"

      - name: Install dependencies
        run: pip install fastapi uvicorn pytest requests

      - name: Run tests
        run: pytest test_api.py
```

### **4️⃣Advanced Error Handling & Logging**
- Use Python's `logging` module to track API requests/responses.
- Store logs in **CloudWatch** or **Elastic Stack**.

### **5️⃣Performance & Load Testing**
- Use **`pytest-benchmark`** or **`locust.io`** to simulate high-traffic conditions.

---

## **Final Thoughts**
In this tutorial, we:
Built a **FastAPI server** with three endpoints  
Wrote **automated tests** using `requests`  
Improved testing with **pytest**  
Discussed **expanding to real-world applications**  

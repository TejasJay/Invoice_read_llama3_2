This Python script leverages **Ollama** and **LLaMA 3.2 Vision** to process invoice images. It extracts structured data using **OCR (Optical Character Recognition)** and organizes it into a predefined format. The extracted information is then either stored as structured JSON or described in a natural language format.

* * *

### **1\. Pulling the LLaMA 3.2 Vision Model**

python

CopyEdit

`#!ollama pull llama3.2-vision`

-   This command (which starts with `#!`) is meant to be run in a shell or script.
-   It **downloads** the `llama3.2-vision` model from **Ollama** if it is not already available.
-   This model is **multimodal**, meaning it can process both **text and images**.
* * *

### **2\. Importing Required Modules**

python

CopyEdit

`from typing import List import ollama from pydantic import BaseModel`

-   `typing.List`: Used to define lists in Pydantic models.
-   `ollama`: The core library that allows us to interact with **LLaMA models**.
-   `pydantic.BaseModel`: Provides **data validation** and **serialization**. It helps structure and validate extracted invoice data.
* * *

### **3\. Defining Data Models (Using Pydantic)**

These models help ensure extracted invoice data is **well-structured and type-safe**.

#### **3.1 Item Model (Represents a single product/service in an invoice)**

python

CopyEdit

`class Item(BaseModel):     name: str     quantity: int     price: float`

-   `name`: The **product/service name**.
-   `quantity`: The **number of units** purchased.
-   `price`: The **cost per unit** (floating-point number).

#### **3.2 Invoice Model (Represents the whole invoice)**

python

CopyEdit

`class Invoice(BaseModel):     invoice_number: str     date: str     vendor_name: str     items: List[Item]     total: float`

-   `invoice_number`: The **unique identifier** for the invoice.
-   `date`: The **date** when the invoice was issued.
-   `vendor_name`: The **business name** issuing the invoice.
-   `items`: A **list of `Item` objects**, each representing a product/service.
-   `total`: The **total amount** charged in the invoice.
* * *

### **4\. Creating a Function to Generate Messages for LLaMA**

python

CopyEdit

`def message(img):     messages=[             {                 'role': 'user',                 'content': """Given an invoice image, Your task is to use OCR to detect and extract text, categorize it into predefined fields.                 Invoice/Receipt Number: The unique identifier of the document.                 Date: The issue or transaction date.                 Vendor Name: The business or entity issuing the document.                 Items: A list of purchased products or services with Name, Quantity and price.""",                 'images': [f"./images/{img}"]             }         ]     return messages`

-   This function **constructs the input message** for the AI model.
-   `img`: The **name of the image file** (e.g., `'your_file.jpg'`).
-   It **requests OCR processing**, asking the model to extract:
    -   Invoice number
    -   Date
    -   Vendor name
    -   List of items (name, quantity, price)
-   The image is **dynamically inserted** using `f"./images/{img}"`.
* * *

### **5\. Extracting Structured Invoice Data**

python

CopyEdit

`res = ollama.chat(     model="llama3.2-vision",     messages=message('your_file.jpg'),     format=Invoice.model_json_schema(),     options={'temperature': 0} )`

-   **`ollama.chat`**: Calls the **LLaMA 3.2 Vision model** for inference.
-   **Arguments passed**:
    -   `model="llama3.2-vision"` → Specifies the model to use.
    -   `messages=message('your_file.jpg')` → Uses the generated prompt from `message()` function.
    -   `format=Invoice.model_json_schema()` → Tells the model to return data in a structured **JSON format**, following the `Invoice` schema.
    -   `options={'temperature': 0}` → Sets **deterministic responses** (no randomness).
**Purpose:**
This **extracts structured invoice data** and formats it as a **JSON object**, ensuring easy parsing and further processing.

* * *

### **6\. Printing the Extracted JSON Data**

python

CopyEdit

`print(res['message']['content'])`

-   Extracts the `"content"` from the model response and **prints the structured invoice data**.
* * *

### **7\. Running a Separate Image Description Query**

python

CopyEdit

`response = ollama.chat(     model='llama3.2-vision',     messages=[{         'role': 'user',         'content': 'Describe What is in this image?',         'images': ['./images/inovice_2.jpg']     }] )`

-   This is a **second Ollama call** to describe an invoice image.
-   It **does not structure the output as JSON**. Instead, it **asks for a textual description**.
* * *

### **8\. Printing the Image Description**

python

CopyEdit

`print(response['message']['content'])`

-   Prints the **natural-language description** of the image.
* * *

## **How the Code Works (Step-by-Step Execution)**

1.  **Pulls the LLaMA 3.2 Vision model** (if not already downloaded).
2.  **Defines data models (`Invoice` & `Item`)** to structure extracted data.
3.  **Constructs a request** to extract invoice details from an image.
4.  **Sends the request to LLaMA 3.2 Vision**, specifying JSON formatting.
5.  **Prints the structured invoice data**.
6.  **Sends another request** to describe an invoice image in **natural language**.
7.  **Prints the description** of the second image.
* * *

## **Enhancements & Next Steps**

✅ **Batch Processing**: Modify the script to process multiple invoices at once.
✅ **Error Handling**: Add exception handling for cases where OCR fails.
✅ **Integration**: Store extracted JSON in a database for further use.
✅ **PDF Support**: Convert PDFs to images and process them.

* * *

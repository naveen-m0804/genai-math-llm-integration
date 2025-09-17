## Integration of a Mathematical Calulations with a Chat Completion System using LLM Function-Calling

### AIM:
To design and implement a Python function for calculating the volume of a cylinder, integrate it with a chat completion system utilizing the function-calling feature of a large language model (LLM).

### PROBLEM STATEMENT:
To calculate Volume of Cylinder by using LLM Function-Calling by passing arguments through function call and display volume in JSON format.

### DESIGN STEPS:

#### STEP 1:
Import required libraries: os, openai, json, math. Load the API key securely using dotenv to access OpenAI.

#### STEP 2:
Create the volume_of_cylinder(radius, height, unit="meter") function.Calculate volume using the formula: 𝑉=𝜋×radius^2×height. Store the result in a dictionary with radius, height, volume, and unit. Return the data in JSON format.

#### STEP 3:
Create a user message to request cylinder volume calculation and show the calculated volume in JSON format. Send a request to openai.ChatCompletion.create(), including:
1. Model: "gpt-3.5-turbo"
2. Messages: User input
3. Functions: Defined function metadata
4. Function Call: Explicitly request "volume_of_cylinder"


### PROGRAM:
```py
import os
import openai
import json
import math

# Load environment variables
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file
openai.api_key = os.environ['OPENAI_API_KEY']

def calculate_cylinder_volume(radius, height):
    """
    Calculates the volume of a cylinder.

    Args:
        radius (float): The radius of the cylinder's base.
        height (float): The height of the cylinder.

    Returns:
        str: A JSON string containing the radius, height, and the calculated volume.
    """
    try:
        # Volume of a cylinder formula: V = pi * r^2 * h
        volume = math.pi * (radius ** 2) * height
        result = {
            "radius": radius,
            "height": height,
            "volume": volume,
        }
    except Exception as e:
        result = {
            "radius": radius,
            "height": height,
            "error": str(e),
        }
    return json.dumps(result)

# Define the function for the LLM to call
functions = [
    {
        "name": "calculate_cylinder_volume",
        "description": "Calculates the volume of a cylinder given its radius and height.",
        "parameters": {
            "type": "object",
            "properties": {
                "radius": {
                    "type": "number",
                    "description": "The radius of the cylinder's base.",
                },
                "height": {
                    "type": "number",
                    "description": "The height of the cylinder.",
                },
            },
            "required": ["radius", "height"],
        },
    }
]

# Create a user message to trigger the function
messages = [
    {
        "role": "user",
        "content": "What is the volume of a cylinder with a radius of 5 and a height of 10?"
    }
]

# Call the ChatCompletion endpoint
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=messages,
    functions=functions
)

# Extract and process the response
response_message = response["choices"][0]["message"]
print("LLM Response Message:")
print(response_message)
print("-" * 20)

# Check if the LLM chose to call the function
if response_message.get("function_call"):
    function_name = response_message["function_call"]["name"]
    function_args = json.loads(response_message["function_call"]["arguments"])
    
    # Verify the function name and call it
    if function_name == "calculate_cylinder_volume":
        print("Calling the 'calculate_cylinder_volume' function...")
        volume_result = calculate_cylinder_volume(
            radius=function_args.get("radius"), 
            height=function_args.get("height")
        )
        print("Function Output:")
        print(volume_result)
        
        # Append the function's output to the messages list
        messages.append(response_message)
        messages.append(
            {
                "role": "function",
                "name": function_name,
                "content": volume_result,
            }
        )
        
        # Call the LLM again with the function's output to get a final response
        print("-" * 20)
        print("Calling LLM with function output for a final response...")
        final_response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=messages
        )
        
        print("Final LLM Response:")
        print(final_response["choices"][0]["message"]["content"])
    else:
        print(f"The LLM called an unexpected function: {function_name}")
else:
    print("The LLM did not call a function.")
    print("Final LLM Response:")
    print(response_message["content"])
```
### OUTPUT:
<img width="1034" height="416" alt="image" src="https://github.com/user-attachments/assets/4fef20cb-ef23-419f-a558-325ae04ca9e8" />

### RESULT:
Thus, The integration of a Mathematical Calulations with a Chat Completion System using LLM Function-Calling is executed successfully.

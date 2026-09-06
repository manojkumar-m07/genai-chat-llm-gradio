## Development and Deployment of a 'Chat with LLM' Application Using the Gradio Blocks Framework

### AIM:
To design and deploy a "Chat with LLM" application by leveraging the Gradio Blocks UI framework to create an interactive interface for seamless user interaction with a large language model.

### PROBLEM STATEMENT:
To create a simple web-based chat application that lets users talk to an AI model in real time using a friendly interface.
### DESIGN STEPS:
### Step 1: 
Import the required libraries such as Gradio, Requests, OS, and python-dotenv for creating the interface, sending API requests, and loading environment variables.
### Step 2:
Load the Hugging Face API key and model endpoint from the .env file to securely connect the application with the Large Language Model.
### Step 3: 
Create a text-generation function that accepts the user's prompt and maximum token value and sends the request to the Hugging Face API.
### Step 4:
Process the API response and extract the generated text. Handle errors such as invalid responses, connection errors, and request timeouts.
### Step 5: 
Design the user interface using the Gradio Blocks framework with a prompt textbox, maximum-token slider, Generate Response button, and output textbox.
### Step 6:
Launch the Gradio application on an available port and test the application by entering different prompts and displaying the generated responses.
### PROGRAM:
```
import os
import requests
import gradio as gr
from dotenv import load_dotenv, find_dotenv


load_dotenv(find_dotenv())

hf_api_key = os.getenv("HF_API_KEY")
endpoint_url = os.getenv("HF_API_FALCOM_BASE")

# Check environment variables
if not hf_api_key:
    raise ValueError("HF_API_KEY is missing from the .env file.")

if not endpoint_url:
    raise ValueError("HF_API_FALCOM_BASE is missing from the .env file.")



def generate_text(prompt, max_tokens):

    if not prompt or not prompt.strip():
        return "Please enter a prompt."

    headers = {
        "Authorization": f"Bearer {hf_api_key}",
        "Content-Type": "application/json"
    }

    data = {
        "inputs": prompt.strip(),
        "parameters": {
            "max_new_tokens": int(max_tokens),
            "temperature": 0.7,
            "return_full_text": False
        }
    }

    try:

        response = requests.post(
            endpoint_url,
            headers=headers,
            json=data,
            timeout=120
        )



        if response.status_code != 200:
            return (
                f"Hugging Face API Error\n\n"
                f"Status Code: {response.status_code}\n\n"
                f"Details: {response.text[:1000]}"
            )


        try:
            result = response.json()

        except ValueError:
            return (
                "The API returned an unexpected response.\n\n"
                f"Response: {response.text[:1000]}"
            )

        if isinstance(result, list) and len(result) > 0:

            if "generated_text" in result[0]:
                return result[0]["generated_text"]

            return str(result[0])

        elif isinstance(result, dict):

         
            if "error" in result:
                return f"API Error: {result['error']}"

            # Handle generated text
            if "generated_text" in result:
                return result["generated_text"]

            return str(result)

        else:
            return str(result)

    except requests.exceptions.Timeout:
        return "Error: The request timed out. Please try again."

    except requests.exceptions.ConnectionError:
        return "Error: Could not connect to the Hugging Face API."

    except Exception as e:
        return f"Error: {str(e)}"


gr.close_all()

with gr.Blocks() as demo:

    gr.Markdown("# 🤖 Chat with LLM")

    gr.Markdown(
        "Enter a prompt and generate a response using "
        "a Hugging Face Large Language Model."
    )

    # Prompt input
    prompt = gr.Textbox(
        label="Enter Your Prompt",
        placeholder="Example: Explain Artificial Intelligence in simple words.",
        lines=4
    )

    # Maximum token slider
    max_tokens = gr.Slider(
        minimum=10,
        maximum=512,
        value=100,
        step=10,
        label="Maximum Tokens"
    )

    # Generate button
    generate_button = gr.Button("Generate Response")

    # Output
    output = gr.Textbox(
        label="LLM Response",
        lines=10
    )

    # Button event
    generate_button.click(
        fn=generate_text,
        inputs=[prompt, max_tokens],
        outputs=output
    )

    # Enter key event
    prompt.submit(
        fn=generate_text,
        inputs=[prompt, max_tokens],
        outputs=output
    )

demo.launch(
    share=True,
    server_port=7861
)
```
### OUTPUT:
<img width="929" height="401" alt="image" src="https://github.com/user-attachments/assets/31b4c807-9c44-435f-a504-37f2ccfa6637" />

<img width="921" height="343" alt="image" src="https://github.com/user-attachments/assets/0a37b14c-e78f-4d8c-9363-c7fece73fde4" />

### RESULT:
Thus, a “Chat with LLM” prototype application was successfully developed using the Gradio Blocks UI framework and a Hugging Face Large Language Model. The application accepts user prompts and displays the generated responses interactively.

# Build-a-Chatbot-for-Your-Data
Create a chatbot for your own pdf file using Flask, and LangChain,

## Setting up and understanding the user interface
Set up the environment by executing the following code:
  pip3 install virtualenv 
  virtualenv my_env # create a virtual environment my_env
  source my_env/bin/activate # activate my_env
  
Run the following commands to retrieve the project:
  git clone https://github.com/ibm-developer-skills-network/wbphl-build_own_chatbot_without_open_ai.git
  mv wbphl-build_own_chatbot_without_open_ai build_chatbot_for_your_data
  cd build_chatbot_for_your_data
  
Installing the requirements for the project:
  pip install -r requirements.txt
  pip install "langchain==0.3.27" "langchain-core==0.3.80" "langchain-community==0.3.31" "langchain-huggingface==0.3.1" "langchain-text-splitters==0.3.11" "huggingface-hub==0.36.0" "tokenizers==0.19.1" "transformers==4.43.3" "sentence-transformers==2.7.0" "requests==2.32.5"
  
Understanding the worker: Document processing and conversation management worker:
  The worker.py is designed to provide a conversational interface that can answer questions based on the contents of a given PDF document.
    Initialization init_llm():
      Setting environment variables: The environment variable for the HuggingFace API token is set.
      Loading the language model: The WatsonX language model is initialized with specified parameters.
      Loading embeddings: Embeddings are initialized using a pre-trained model.
    Document processing process_document(document_path):
      This function is responsible for processing a given PDF document.      
        Loading the document: The document is loaded using PyPDFLoader.
        Splitting text: The document is split into smaller chunks using RecursiveCharacterTextSplitter.
        Creating embeddings database: An embeddings database is created from the text chunks using Chroma.
        Setting Up the RetrievalQA chain: A RetrievalQA chain is set up to facilitate the question-answering process. This chain uses the initialized language model and the embeddings database to answer questions based on the processed document.
    User prompt processing process_prompt(prompt):
      This function processes a user's prompt or question.      
        Receiving user prompt: The system receives a user prompt (question).
        Querying the model: The model is queried using the retrieval chain, and it generates a response based on the processed document and previous chat history.
        Updating chat history: The chat history is updated with the new prompt and response.

  Note: This access method is exclusive to this Cloud IDE environment. If you are interested in using the model/API outside this environment (e.g., in a local environment), detailed instructions and further information are available in this [tutorial](https://medium.com/the-power-of-ai/ibm-watsonx-ai-the-interface-and-api-e8e1c7227358)

## Running the app in CloudIDE:
  Run the server: python3 server.py
  Demo: 
  <img width="1393" height="884" alt="image" src="https://github.com/user-attachments/assets/bf2d406f-e9ab-4717-9102-9f1e71fdc3e4" />


## Using HuggingFace API for the worker
  Create the base LLM using HuggingFaceEndpoint : The HuggingFaceEndpoint object connects to a HuggingFace-hosted model using the Inference API.The HuggingFaceEndpoint object is created with the specified repo_id and additional parameters like temperature, max_new_tokens and your your HuggingFace API tokento control the behavior of the model. Here you can find more examples.
  Wrap the base LLM with ChatHuggingFace : The ChatHuggingFace wrapper converts the base LLM into a chat-compatible model, allowing it to work seamlessly with LangChain chat-based chains (such as RetrievalQA).
  The embeddings are initialized using a class called HuggingFaceEmbeddings, pre-trained model named sentence-transformers/all-MiniLM-L6-v2. A list of leaderboards of embeddings are available here. This embedding model has shown a good balance in both performance and speed.
  The model uses the specified device (CPU or GPU) for computation.

  Solution:
    '# Function to initialize the language model and its embeddings
        def init_llm():
        global llm_hub, embeddings
        
        os.environ["HUGGINGFACEHUB_API_TOKEN"] = "YOUR API KEY"
        
        base_llm = HuggingFaceEndpoint(
            repo_id="meta-llama/Llama-3.1-8B-Instruct",
            task="text-generation",
            huggingfacehub_api_token=os.environ["HUGGINGFACEHUB_API_TOKEN"],
            temperature=0.1,
            max_new_tokens=600,
        )
        
        # wrap with ChatHuggingFace
        llm_hub = ChatHuggingFace(llm=base_llm)
        
        embeddings = HuggingFaceEmbeddings(
            model_name="sentence-transformers/all-MiniLM-L6-v2",
            model_kwargs={"device": DEVICE},
        )
    '
  You also need to insert your LLM API key. Here's a demonstration to show you how to get your API key.

  Initialize HuggingFace API key from your account with the following steps:
  
  Go to the https://huggingface.co/
  Log in to your account (or sign up free if it is your first time)
  Go to Settings -> Access Tokens -> click on New Token (refer image below)
  Select either read or write option and copy the token

  <img width="877" height="553" alt="image" src="https://github.com/user-attachments/assets/8c51a1f2-4c01-42ee-9f51-c1a4d90aa25a" />
  <img width="861" height="611" alt="image" src="https://github.com/user-attachments/assets/442e4682-6f12-4674-8235-a862c89fea29" />

  To implement your chatbot, you need to run the server.py file, first.
    python3 server.py
  Demo:
    <img width="757" height="681" alt="image" src="https://github.com/user-attachments/assets/e98d0700-50bf-4caa-8539-af68624c9c4e" />


## Understanding the server
  The server is how the application will run and communicate with all of your services.
  This project uses Flask to handle the backend of your chatbot. This means that you will be using Flask to create routes and handle HTTP requests and responses. When a user interacts with the chatbot through the front-end interface, the request will be sent to the Flask backend. Flask will then process the request and send it to the appropriate service.
  The server.py file consists of 3 functions which are defined as routes, and the code to start the server.

  When a user tries to load the application, they initially send a request to go to the / endpoint. They will then trigger this index function and execute the code above. Currently, the returned code from the function is a render function to show the index.html file which is the frontend interface.

  The second and third routes are what will be used to process all requests and handle sending information between the application. The process_document_route() function is responsible for handling the route when a user uploads a PDF document, processing the document, and returning a response. The process_message_route() function is responsible for processing a user's message or query about the processed document and returning a response from the bot.
  
  Finally, the application is started with the app.run command to run on port 8080 and the host as 0.0.0.0 (a.k.a. localhost).

## Explaining JavaScript file
  The JavaScript file is responsible for managing the user interface and interactions of a chatbot application. It is located in static folder. The main components of the file are as follows:

  Message processing: The function processUserMessage(userMessage) sends a POST request to the server with the user's message and waits for a response. The server processes the message and returns a response that is displayed in the chat window.
  Loading animations: The functions showBotLoadingAnimation() and hideBotLoadingAnimation() show and hide a loading animation while the server is processing a message or document.
  Message display: The functions populateUserMessage(userMessage, userRecording) and populateBotResponse(userMessage) format and display user messages and bot responses in the chat window.
  Input cleaning: The function cleanTextInput(value) cleans the user's input to remove unnecessary spaces, newlines, tabs, and HTML tags.
  File upload: The event listener for $("#file-upload").on("change", ...) handles the file upload process. When a file is selected, it reads the file data and sends it to the server for processing.
  Chat reset: It provides a way to reset the chat, clearing all messages and starting over with the initial bot greeting.
  Light/Dark mode switch: The event listener for $("#light-dark-mode-switch").change(...) allows the user to switch between light and dark modes for the chat interface.

## Docker file: 
Docker allows for the creation of “containers” that package an application and its dependencies together. This allows the application to run consistently across different environments, as the container includes everything it needs to run. Additionally, using a Docker image to create and run applications can simplify the deployment process, as the image can be easily distributed and run on any machine that has Docker. This easy distribution of image ensures that the application runs in the same way in development, testing, and production environments.

'  # Use Python 3.10 base image
FROM python:3.10

# Set working directory
WORKDIR /app

# Copy all project files
COPY . .

# Install dependencies from requirements.txt
# Then install specific versions of langchain, huggingface, transformers, etc.
RUN pip install --no-cache-dir -r requirements.txt \
    && pip install --no-cache-dir \
       langchain==0.3.27 \
       langchain-core==0.3.80 \
       langchain-community==0.3.31 \
       langchain-huggingface==0.3.1 \
       langchain-text-splitters==0.3.11 \
       huggingface-hub==0.36.0 \
       tokenizers==0.19.1 \
       transformers==4.43.3 \
       sentence-transformers==2.7.0 \
       requests==2.32.5

# Expose port if needed (Flask default 8000)
EXPOSE 8000

# Run the server
CMD ["python", "-u", "server.py"]'

Commands:
  docker build --no-cache -t build_chatbot_for_your_data .
  docker run -p 8000:8000 build_chatbot_for_your_data

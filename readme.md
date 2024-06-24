# **Project Documentation: Building a Customized Chat Bot for Science-Related Questions**

# Introduction
Welcome to the documentation of the customized chat bot developed for science-related questions aka the science bot. This document outlines the journey and key components of the project, from requirements to implementation. The outcome of this project developed is to handle scientific questions intelligibly by a bot and provide the best possible answer to it.

There are 2 setups in this project that includes a backend setup and a frotend setup. 

# How to run?
**Download and install all the dependencies –**

•	I have provided the requirement.txt file

•	Download the Llama quantized model from https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGML/blob/main/llama-2-7b-chat.ggmlv3.q8_0.bin 

•	We can run the server on port 5500 locally by using the start.sh script or within a Docker container using the Dockerfile. (Backend setup - users can access the bot through backend at port 5500)

•	To run the UI interface built with the help of Chainlit go to ur environment terminal and type - "chainlit run test,py -w". (Frontend setup - users can access the bot through this instruction)

# Video Link
https://drive.google.com/file/d/1t1LANxfBsGpJ9Akjfkvk12JnSDgvWatb/view?usp=sharing 

# Directory Structure
The project structure is organized as follows:

* main.py: The main application file handling the Flask server (introduced by me), request processing, and response generation.
* simple_text.py: Defines the SimpleText class and its schema using Marshmallow.
* ignite.py: Launches the Flask server.
* datastore/: Contains state information (state.json) and individual query data (query_id.json) in a structured format.
* vectorstore/: (additional directory) Contains the generated embeddings of the texts in vector format.
* preprocessing.py: This file was introduced by me (additional file) for document loading and text splitting contributing towards overall text preprocessing of data.
* test.py: This file was introduced by me (additional file). This file contains the logic behind our large language model in which the input query is processed to obtain the desired results through our model.

# Implementation Steps

**1. Understanding the Test**

The initial step involved thoroughly understanding the provided test description, including mission, ground rules, and deployment options. I had to first go through openfabrics documentation (https://docs.openfabric.ai/developer-tools/index/) to understand the underlying data structures, classes and functions inside the project directory.

**2. Framework Selection**

The project utilizes the Openfabric PySDK for interacting with the Openfabric platform and handling execution context.

**3. Text Preprocessing**

For this step I have introduced a new file known as preprocessing.py. In this file, I have leveraged the might of Langchain. Firstly, we load the data that is stored in the data/ folder in pdf format using PyPDFLoader and DirectoryLoader. Next, we use RecursiveCharacterTextSplitter from langchian to split the text into chunks. Next, after splitting the text we pass it into the sentence-transformer model from the HuggingFaceEmbeddings dependency which generates embeddings for our text data.
To store these embeddings in text format I used FAISS module.

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/0bf1b108-8cb8-4437-9a0c-3d6fe5371df0)
   
To generate these vector stores before executing our script we need to run this preprocessing.py file to generate the vector stores.
Output - 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/9c43420a-0c35-4654-b587-7f5ec01ee17c)

These files must be generated.

**4. Coding the execute Function**

The core of the application is the execute function in main.py. This function processes incoming requests, handles each query, and generates appropriate responses. 
To design this function firstly I introduced another function called handle_request(). Using flask I developed this function to handle incoming request to the server and pass it onto the execute function to collect the response.
 
Code snippet -  

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/bdceefac-bb98-4938-8daa-aee64267a4a9)

Here we can see when we pass data into the execute function we declare a none ray class and an empty state class with default values being empty.
Next, I wrote a function to generate 32-character length of random alphanumeric data for our session and query ids.

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/da8410ca-5973-47d2-8165-95b1647023ff)

Finally, I dove into the execute function in which I setup ray class for each query having unique query and session ids. Then I iterated through each query in the request consisting of a list of queries. For each query processing is done and response is collected by passing it onto the final_result function.

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/51120619-9d43-411e-a073-39c80337adb5)
 
The datastore also gets updated in this function in which a unique query_id.json file is stored and the state.json file is updated.
Finally the function returns the output in a dictionary format of the SchemaUtilclass through simpleText function.

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/590c134b-ef07-4254-ab6d-6c10a074280f)
 
# 5. Integration with Custom Model (test.py)

The model.py file contains the implementation of the custom chat bot model. It leverages language models for question answering, embeddings, and a vector store for efficient retrieval.
So, when the final_result() function in execute function of main.py file is run we come to this file, the test.py file.

**Step 1 – Defining a custom prompt template and a function to set it up**

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/952a0a8e-1c72-42f2-a211-bf51bd994871)
![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/37be97e0-8912-4cd4-b2bf-d71de798ed57)
 
**Step 2 – Defining the retrieval QA chain function**

In this function a chain of events is run inorder to retrieve the correct information for our response. This can be better explained with the diagram given below
Diagram – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/5307a2ef-8937-4df3-b402-08253c9a5868)

Here you can see that on passing the query the qa retrieval function gets activated. It goes into our docs or data that we have provided. Then after getting multiple sources. It uses LLMChain model to retrieve the useful information to form the response and finally using ctransformers we load the answer.

**Step 3 – Loading the model**

For this project I have used the quantized version of the Llama 2, 7 billion bytes quantization model. This quantized version was downloaded from “TheBloke/Llama-2-7B-Chat-GGML”.  

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/a66b3f43-e8ef-428d-8363-dcb6277278ba)

Defining the Question Answer model function

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/8978c734-146c-4536-8a25-2f021fc72589)

Combining everything to form the output function

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/31248254-1293-4016-9adf-25557e54924a)

# 6. Error Handling
Error handling was implemented within the execute function to gracefully manage errors during query processing. The messages attribute in the ray class was updated with error details.

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/583a2861-8ccf-407f-8d92-d2d5d446796d)

# 7. State Management

The state.json file in the datastore/ directory is used to track the status of each query, including queued, requested, and completed queries. The state is updated at various stages of query processing.

# 8. Output Storage

The output of each query, along with relevant information, is stored in a separate file (query_id.json) within the datastore/queries/ directory.

# 9. Output

Output – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/746c44e7-559b-4288-a80c-f55cae428fb1)

In this snippet you can see we have used postman to generate the response for 3 queries passed into the request body in json format.
 
# 10. UI Interface 

By leveraging the functionalities of chainlit I have developed a custom code for our bot’s user interface. 
To see a working UI through chainlit go to ur environment terminal and type - "chainlit run test,py -w"

Code snippet – 

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/e0aef910-dcf3-4841-9943-6f8ecfc0d52a)

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/73f08ef6-1fb5-41aa-801f-4ead24073152)

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/03a651a7-b52d-4cb0-bb11-c43056c2fe52)

![image](https://github.com/Prats13/Personalised_Science_Chat_Bot/assets/93511663/d3f06d34-3eb8-47b4-9125-f59ea309eb52)

# Usage
The application can be initiated locally using the provided start.sh script or within a Docker container using the Dockerfile.

# Future Scope 
As I developed this model in the span of one week. I would not dare say that it is perfect as still there are multiple changes and upgrades that can be implemented. The speed of response generation is a factor. Then we can work upon the efficiency of the answers as couple of times I have received incorrect or out of context answers for some question which required multiple runs.

# Conclusion
The customized chat bot successfully addresses the test requirements, providing a comprehensive solution for science-related questions. The project demonstrates effective use of the Openfabric PySDK, integrates a custom model, handles errors, and manages state and output storage efficiently.

# Important links:
Docs of openfabric_pysdk - https://docs.openfabric.ai/developer-tools/index/
Langchain docs - https://python.langchain.com/docs/get_started/introduction
Chainlit docs – 
https://github.com/Chainlit/chainlit
Ctransformers docs – 
https://github.com/marella/ctransformers

# Contact:
Feel free to reach out if you have any questions or need further clarification.
Email: pratyushpatra13@gmail.com
Phone No.: +91 9612276838
End of Documentation

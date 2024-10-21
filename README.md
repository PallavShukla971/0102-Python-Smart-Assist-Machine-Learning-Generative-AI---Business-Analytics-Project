# 0102-Python-Smart-Assist-Machine-Learning-Generative-AI-Business-Analytics-Project

Python Scripts for project.

Steps:
______________________________________________________
Step 1: Install the required libraries.
- pip install PyPDF2 pdfplumber azure-storage-blob

______________________________________________________
Step 2: Extract content from PDF files.

Here's Python code for offline storage:

______________________________________________________
import os
import PyPDF2
import pdfplumber
import sqlite3  # For local storage in an SQLite DB

# Specify the folder containing the PDFs
folder_path = './pdf_folder'

# Set up a local SQLite database to store the extracted content
conn = sqlite3.connect('sop_documents.db')
cursor = conn.cursor()

# Create table for storing PDF content
cursor.execute('''
    CREATE TABLE IF NOT EXISTS sop_documents (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        filename TEXT,
        content TEXT
    )
''')
conn.commit()

# Function to extract content from PDF
def extract_pdf_content(pdf_file):
    content = ""
    with pdfplumber.open(pdf_file) as pdf:
        for page in pdf.pages:
            content += page.extract_text()
    return content

# Loop through each PDF in the folder
for filename in os.listdir(folder_path):
    if filename.endswith('.pdf'):
        pdf_path = os.path.join(folder_path, filename)
        content = extract_pdf_content(pdf_path)

        # Store content in the SQLite database
        cursor.execute("INSERT INTO sop_documents (filename, content) VALUES (?, ?)", (filename, content))
        conn.commit()

# Close the connection after storing all PDFs
conn.close()
print("Documents stored locally in SQLite database.")


______________________________________________________


In this method:

PDF content is extracted locally using pdfplumber.
All extracted text is stored in an SQLite database for local access and processing.
Method 2: Uploading to Azure Storage (When Online)
Once you’re ready to go online, the stored PDF files or the extracted content can be uploaded to Azure Blob Storage. Here’s the code to upload files to Azure:

Step 3: Set up Azure Blob Storage SDK. First, ensure you have an Azure Storage Account set up and note down the connection string.
______________________________________________________

from azure.storage.blob import BlobServiceClient

# Azure Storage connection string
connection_string = "your_azure_storage_connection_string"

# Create a blob service client
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Name of the container in Azure Storage
container_name = 'sop-documents'

# Ensure the container exists
try:
    container_client = blob_service_client.create_container(container_name)
except Exception as e:
    print("Container already exists or encountered an error:", e)

# Function to upload a file to Azure Blob Storage
def upload_to_azure(file_path, blob_name):
    try:
        blob_client = blob_service_client.get_blob_client(container=container_name, blob=blob_name)

        # Upload the file
        with open(file_path, 'rb') as file_data:
            blob_client.upload_blob(file_data, overwrite=True)

        print(f"Uploaded {file_path} to Azure as {blob_name}")

    except Exception as e:
        print(f"Error uploading {file_path}: {e}")

# Loop through PDFs again for Azure upload
for filename in os.listdir(folder_path):
    if filename.endswith('.pdf'):
        pdf_path = os.path.join(folder_path, filename)
        upload_to_azure(pdf_path, filename)

______________________________________________________

In this method:

You upload the PDF files directly to Azure Blob Storage using Azure Storage Blob SDK.
Ensure your storage connection string is configured correctly.
2. Fully Offline Solution with Later Upload to Azure
If you prefer to keep everything offline, including the storage in files rather than SQLite, you can simply save extracted content into .txt files locally.
______________________________________________________

# Function to save extracted content to text files
def save_to_text_file(filename, content):
    with open(f"./extracted/{filename}.txt", "w", encoding="utf-8") as file:
        file.write(content)

# Extract and save each PDF as a text file
for filename in os.listdir(folder_path):
    if filename.endswith('.pdf'):
        pdf_path = os.path.join(folder_path, filename)
        content = extract_pdf_content(pdf_path)
        save_to_text_file(filename, content)

______________________________________________________
Later, when the system goes online, you can upload the .txt files or the original PDF files to Azure Blob Storage.

Summary of Approaches:
First method stores extracted PDF content locally in an SQLite database.
Second method uploads PDF content to Azure Blob Storage once the system is online, ensuring a hybrid of local and cloud storage.
Would you like more help on connecting to Azure or fine-tuning the document indexing?
______________________________________________________

______________________________________________________

______________________________________________________

______________________________________________________

______________________________________________________

______________________________________________________

______________________________________________________

______________________________________________________


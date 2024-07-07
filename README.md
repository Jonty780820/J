import pytesseract
from pdf2image import convert_from_path
import re
from datetime import datetime
import spacy

# Load SpaCy model
nlp = spacy.load("en_core_web_sm")

# Convert PDF to images
pages = convert_from_path('your_document.pdf', 300)

# OCR the images
text = ''
for page in pages:
    text += pytesseract.image_to_string(page)

# Extract dates and service providers
date_pattern = r'\b(\d{1,2}[/-]\d{1,2}[/-]\d{2,4})\b'
dates = re.findall(date_pattern, text)
dates = [datetime.strptime(date, '%d/%m/%Y') for date in dates]

# Use SpaCy to identify service providers
doc = nlp(text)
providers = [ent.text for ent in doc.ents if ent.label_ == "ORG"]

# Sort documents by date
documents = list(zip(dates, providers))
documents.sort(key=lambda x: x[0])

# Output sorted documents
sorted_text = ''
for date, provider in documents:
    sorted_text += f'Date: {date.strftime("%d/%m/%Y")}\nProvider: {provider}\n\n'

with open('sorted_documents.txt', 'w') as f:
    f.write(sorted_text)
    

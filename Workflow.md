# Extracting tables/data from a PDF 

## The approach: an AI assisted-method
We will ask an AI tool help us complete the task by giving us some code that we can run ourselves in Google Colab. I will explain what Google Colab is.

## The tools we will use for this task

### 1. Google colab
Google Colaboratory is an app in Google Drive used for code editing (writing and running you Python code). 
It is a free, cloud-based Jupyter Notebook environment provided by Google, allowing users to write and execute Python code through the browser, especially suited for machine learning, data analysis.
There are other code editing tools such as Visual Code Studio. But we will use Colab because it requires no installation.

### 2. Camelot (for 


### 3. img2table
For image-based PDF (scanned pdf). Here you can find the document I used for this exercice. It is called "docdoc.pdf". (I printed an excel file and then scanned it to get an image-based PDF).
img2table is a Python Library for table identification and extraction.

The python code
```
!pip install img2table
!pip install pytesseract
!apt-get install tesseract-

from img2table.document import PDF
from img2table.ocr import TesseractOCR

pdf_path = “docdoc.pdf" # replace "docdoc.pdf" by the name of you actual pdf file. Here our pdf is called "docdoc.pdf".

ocr = TesseractOCR(n_threads=1, lang="eng")  # Use lang="fra" if you document is in french.

doc = PDF(pdf_path, detect_rotation=True)  # Detect_rotation is used to detect biased or rotated documents
extracted_tables = doc.extract_tables(
   ocr=ocr,
    implicit_rows=False,
    borderless_tables=True,
   min_confidence=50
)

for page, tables in extracted_tables.items():
    print(f"Page {page}: found {len(tables)} table(s)”)
    for i, table in enumerate(tables):
        df = table.df
        print(df.head())
        df.to_csv(f”extracted_table_page{page}_{i}.csv”, index=False)
        df.to_excel(f"extracted_table_page{page}_{i}.xlsx", index=False)

```

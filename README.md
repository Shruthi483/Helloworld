# Helloworld
import boto3
import io
import PyPDF2  # Import PyPDF2 for handling PDF text extraction
# Set the AWS region
REGION = 'us-east-1'  # Replace with your region
# Initialize clients for S3 with region
s3 = boto3.client('s3', region_name=REGION)
def extract_text_with_pypdf2(bucket_name, key):
    try:
        print(f"Extracting text from file: {key} using PyPDF2")
        
        # Download the PDF file from S3
        response = s3.get_object(Bucket=bucket_name, Key=key)
        pdf_content = response['Body'].read()
        
        # Open the PDF file with PyPDF2
        pdf_reader = PyPDF2.PdfReader(io.BytesIO(pdf_content))
        
        # Try to extract text from each page
        for page_num in range(len(pdf_reader.pages)):
            page = pdf_reader.pages[page_num]
            text = page.extract_text()
            
            # If any page has selectable text, it's a non-scanned PDF
            if text and text.strip():  # If text exists and is not just whitespace
                return True
        
        # No selectable text found, this is likely a scanned PDF
        return False
    except Exception as e:
        print(f"Error in extract_text_with_pypdf2: {e}")
        return False

def classify_pdf_with_pypdf2(bucket_name, key):
    """
    Classifies a PDF as 'Scanned PDF' or 'Non-Scanned PDF' based on text extraction using PyPDF2.
    """
    print(f"Classifying PDF using PyPDF2: {key}")
    try:
        # Check if PyPDF2 can extract text from the PDF
        text_extracted = extract_text_with_pypdf2(bucket_name, key)

        if text_extracted:
            return "Non-Scanned PDF"
        else:
            return "Scanned PDF"
    except Exception as e:
        print(f"Error in classify_pdf_with_pypdf2: {e}")
        return "An error occurred during classification"

def lambda_handler(event, context):
    """
    Lambda function entry point.
    """
    bucket_name = event.get('bucket_name', 'test-classification5')  # Replace with your S3 bucket name
    key = event.get('key')  # Replace with the key of the PDF file to classify

    if not key:
        return {
            'statusCode': 400,
            'body': 'Missing "key" in event.'
        }
    
    print(f"Starting classification for file: {key}")
    classification = classify_pdf_with_pypdf2(bucket_name, key)

    return {
        'statusCode': 200,
        'body': json.dumps({"file": key, "classification": classification})
    }

# If running locally, use the following
if __name__ == "__main__":
    # Example usage
    bucket_name = 'test-classification5'  # Replace with your S3 bucket name
    key = 'example.pdf'  # Replace with the key of the PDF file to classify
    classification = classify_pdf_with_pypdf2(bucket_name, key)
    print(f'File name: {key}\nFile Type: {classification}\n')

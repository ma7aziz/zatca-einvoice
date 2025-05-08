# ZATCA E-Invoice Generator

This application creates ZATCA-compliant electronic invoices according to Saudi Arabia's e-invoicing regulations (Fatoora).

## Overview

The ZATCA E-Invoice Generator creates structured XML documents that comply with the technical specifications required by the Saudi Arabian tax authority. The application handles:

1. XML document creation following UBL 2.1 standards
2. Digital signatures using ECDSA P-256
3. QR code generation for simplified invoice validation
4. All required ZATCA business rules compliance

## Requirements

- Python 3.7+
- Required Python packages:
  - `pycryptodome` - For digital signature creation
  - `lxml` - For XML processing
  - `qrcode` - For QR code generation
  - `pillow` - For image processing (QR code generation)

## Installation

1. Clone the repository:
```
git clone https://github.com/yourusername/zatca-einvoice.git
cd zatca-einvoice/e-invoice\ mini\ app
```

2. Install dependencies:
```
pip install pycryptodome lxml qrcode pillow
```

## How It Works

### 1. Invoice Data Structure

The process begins by creating a simplified invoice data structure with essential information:
- Seller details (name, VAT number)
- Invoice total amounts (with and without VAT)
- Line items with prices, quantities, and descriptions
- Invoice date and UUID

### 2. Digital Signature Process

The app follows ZATCA's signing requirements:
1. **Key Generation**: ECDSA P-256 key pairs are generated for signing
2. **Invoice Serialization**: The invoice data is converted to a string representation
3. **Hashing**: SHA-256 algorithm is used to create a hash of the invoice data
4. **Digital Signing**: The hash is signed using the private key
5. **Encoding**: The signature is base64-encoded for inclusion in the XML

### 3. XML Generation

The app creates a structured XML document following strict UBL 2.1 standards:
1. The root `<Invoice>` element with proper namespaces
2. UBL extensions for the digital signature
3. Common invoice header elements (ID, date, type code)
4. Seller and buyer information with address details
5. Tax breakdown following ZATCA requirements
6. Line items with detailed product information
7. Monetary totals and tax calculations

### 4. QR Code Generation

A QR code is generated according to ZATCA specifications:
1. Combines seller VAT, timestamp, total, VAT amount, and hash
2. The hash is created from these elements using SHA-256
3. The final QR contents are base64-encoded for inclusion in the invoice
4. The QR code can be used to quickly validate the invoice

### 5. XML Validation Requirements

The e-invoice must meet specific structural requirements:
- Elements must appear in the correct sequence
- Required fields must be present and correctly formatted
- Address fields must include all mandatory components
- Tax calculations must be accurate and properly structured

## Using the Application

### Basic Usage

```python
from zatca_app import ZatcaInvoice

# Create ZATCA invoice processor
zatca = ZatcaInvoice()

# Define line items
items = [
    {
        'id': 1, 
        'name': 'Product A', 
        'price': 500.00, 
        'quantity': 2, 
        'unit_code': 'PCE'
    }
]

# Generate a complete invoice
invoice_xml = zatca.generate_complete_invoice(
    seller_name="ABC Company",
    seller_vat="310000000000003",
    total_amount=1150.00,
    vat_amount=150.00,
    items=items,
    output_path="zatca_invoice.xml"
)
```

### Advanced Usage with Previous Invoice Hash

For establishing an invoice chain (as required by ZATCA):

```python
# Generate first invoice
first_invoice_xml = zatca.generate_complete_invoice(
    seller_name="ABC Company",
    seller_vat="310000000000003",
    total_amount=1150.00,
    vat_amount=150.00,
    items=items,
    output_path="invoice_1.xml"
)

# Calculate hash of first invoice for chain
import hashlib
import base64
first_invoice_hash = base64.b64encode(
    hashlib.sha256(first_invoice_xml.encode('utf-8')).digest()
).decode('utf-8')

# Generate second invoice with reference to first
second_invoice_xml = zatca.generate_complete_invoice(
    seller_name="ABC Company",
    seller_vat="310000000000003",
    total_amount=2300.00,
    vat_amount=300.00,
    items=items,
    output_path="invoice_2.xml",
    previous_invoice_hash=first_invoice_hash
)
```

## Key Components Explained

### 1. Invoice Counter (ICV)

Each invoice contains an Invoice Counter Value (ICV) that must be sequential for each seller. The ICV is included in AdditionalDocumentReference with ID="ICV".

### 2. Previous Invoice Hash (PIH)

ZATCA requires each invoice to reference the hash of the previous invoice to establish a chain. This reference is stored in AdditionalDocumentReference with ID="PIH".

### 3. Saudi National Address Requirements

Both seller and buyer addresses must include specific components according to Saudi national addressing standards:
- Street name
- Building number
- District/neighborhood
- City name
- Postal code
- Country code

### 4. Tax Breakdown

The invoice includes detailed tax calculations:
- Document currency tax total (with subtotals)
- Tax currency total (without subtotals)
- Line item tax details

## Error Handling

The application provides detailed error messages for common issues:
- Invalid XML structure
- Missing required fields
- Incorrect tax calculations
- Signature generation failures

## Compliance

This application complies with:
- ZATCA E-Invoice XML specification
- UBL 2.1 standards
- Saudi Electronic Invoice Regulations
- ZATCA business rules and validations

## License

## License

MIT License

Copyright (c) 2023 

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Contact

EMAIL : ma7moud.aelaziz@gmail.com
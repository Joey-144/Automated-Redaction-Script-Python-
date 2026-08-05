import fitz  # PyMuPDF
import re

# ==========================================
# 1. FILE PATH CONFIGURATION
# ==========================================
# Update these paths to point to your target files
INPUT_PDF = r"C:\path\to\your\input_document.pdf"
OUTPUT_PDF = r"C:\path\to\your\output_redacted.pdf"

# ==========================================
# 2. TARGET TEXT & REGEX PATTERNS
# ==========================================

# Exact names or specific text strings to redact
EXACT_TEXT = [
    "John Doe",
    "Sample Name",
    "Specific Keyword"
    # Add additional exact phrases or surnames here
]

# Dynamic regex patterns to target monetary values, dates, etc.
PATTERNS = [
    # 1. Monetary values (e.g., $50.00, $1,520,015.47, -$100.00)
    r"-?\$\s*\d{1,3}(?:,\d{3})*(?:\.\d{2})?",
    
    # 2. Date expressions starting with "As of [date]"
    r"(?i)\bas\s+of\s+(?:\d{1,2}[/-]\d{1,2}[/-]\d{2,4}|[A-Za-z]+\s+\d{1,2}(?:st|nd|rd|th)?,?\s*\d{4})\b"
]

# Pattern for 7-digit order numbers (targets only digits, preserving surrounding spaces)
ORDER_NUMBER_PATTERN = r"\b\d{7}\b"


def apply_redactions(input_path, output_path):
    """
    Opens a PDF, searches for specified text and regex patterns, 
    applies black redaction annotations, and permanently purges 
    the underlying text bytes from the file.
    """
    doc = fitz.open(input_path)

    for page in doc:
        # 1. Redact exact text strings
        for text in EXACT_TEXT:
            matches = page.search_for(text)
            for rect in matches:
                page.add_redact_annot(rect, fill=(0, 0, 0))

        # 2. Extract page text and search regex patterns (currency, dates)
        page_text = page.get_text("text")
        for pattern in PATTERNS:
            for match in re.finditer(pattern, page_text):
                matched_str = match.group(0)
                matches = page.search_for(matched_str)
                for rect in matches:
                    page.add_redact_annot(rect, fill=(0, 0, 0))

        # 3. Redact isolated 7-digit order numbers
        for match in re.finditer(ORDER_NUMBER_PATTERN, page_text):
            order_num = match.group(0)
            matches = page.search_for(order_num)
            for rect in matches:
                page.add_redact_annot(rect, fill=(0, 0, 0))

        # 4. Permanently remove underlying text content and stream data
        page.apply_redactions()

    # Save cleaned file with stream compression and garbage collection
    doc.save(output_path, garbage=4, deflate=True)
    doc.close()
    print(f"Redaction complete! Output saved to:\n{output_path}")


if __name__ == "__main__":
    apply_redactions(INPUT_PDF, OUTPUT_PDF)

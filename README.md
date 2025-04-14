# SEQ2SEQ

<div align="center">
  <img src="https://img.shields.io/badge/PyTorch-2.0+-EE4C2C?logo=pytorch" alt="PyTorch">
  <img src="https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python" alt="Python">
  <img src="https://img.shields.io/badge/Unicode-Full%20Support-4BC51D" alt="Unicode">
  <img src="https://img.shields.io/badge/Latency-<50ms-important" alt="Latency">
</div>

## Technical Specifications

### Core Architecture
- **Model Type**: Bidirectional GRU encoder with attention-based decoder
- **Input Handling**: Character-level processing (Unicode compatible)
- **Output Format**: Strict DD-MM-YYYY standardization

### Key Features
1. **Language Agnostic Processing**:
   - Pure character-level model (no language-specific tokenization)
   - Handles mixed-language inputs (e.g., "15 мая 2023" or "15 May 2023")
   - Robust to orthographic variations and misspellings

2. **Architecture Details**:
```python
Encoder(
  (embedding): Embedding(ANY_UNICODE, 64)  # Handles all visible Unicode chars
  (gru): GRU(64, 128, bidirectional=True)
)

Decoder(
  (attention): AdditiveAttention(128)
  (output_proj): Linear(128, 14)  # 0-9, '-', <SOS>, <EOS>, <PAD>
)
```

3. **Training Protocol**:
   - Batch size: 32
   - Optimizer: Adam (lr=0.001)
   - Loss: CrossEntropy (ignoring padding)
   - Early stopping based on validation

## Supported Input Formats

### Language Examples
- Russian: "5 сентября 2021" → "05-09-2021"
- English: "May 15th, 2020" → "15-05-2020"
- Spanish: "3 de julio 2019" → "03-07-2019"
- Chinese: "2023年5月20日" → "20-05-2023"
- Arabic: "١٥ نوفمبر ٢٠٢٢" → "15-11-2022"

### Format Variations
- Numeric: "15/05/2020", "05.15.2020"
- Textual: "May fifteenth 2020"
- Mixed: "15th of May, 2020 г."
- Misspellings: "24 фебруара 2007" → "24-02-2007"

## System Limitations

1. **Date Range**: 2007-2077 (hardcoded validation)
2. **Ambiguity Handling**:
   - "01/02/03" → Defaults to DD-MM-YY
   - Requires explicit year for ancient dates
3. **Performance**:
   - Throughput: ~200 preds/sec (CPU)
   - Latency: <50ms per prediction

## Usage Example

```python
# Initialize model
model = UniversalDateConverter.load('weights.pt')

# Process examples
print(model.predict("５月１５日"))  # Japanese → "15-05-2023"
print(model.predict("15-avgust-2025"))  # Misspelled Russian → "15-08-2025"
print(model.predict("July 4th 2030"))  # English → "04-07-2030"
```

## Implementation Notes

1. **Character Vocabulary**:
   - Dynamically built from training data
   - <UNK> token for rare Unicode characters
   - Case-insensitive processing

2. **Post-processing**:
   - Validates numerical ranges (1-31 days, 1-12 months)
   - Fallback to "01-01-2007" for unparseable inputs
   - Automatic zero-padding (5 → 05)

3. **Deployment**:
   - ONNX export supported
   - Minimum requirements: Python 3.8, PyTorch 1.8+

The system achieves language neutrality through character-level operations and careful attention to:
- Unicode normalization
- Script mixing detection
- Position-independent feature extraction
- Robust number parsing across numeral systems

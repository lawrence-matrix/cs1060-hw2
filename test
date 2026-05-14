import pytest
import json
from index import app

@pytest.fixture
def client():
    """Configures the Flask app for testing."""
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_decimal_to_binary(client):
    """Tests basic valid conversions (Decimal to Binary)."""
    payload = {"input": "10", "inputType": "decimal", "outputType": "binary"}
    response = client.post('/convert', json=payload)
    data = json.loads(response.data)
    
    assert response.status_code == 200
    assert data['result'] == "1010"
    assert data['error'] is None

def test_base64_endianness_bug(client):
    """
    Tests base64 conversions to catch the mandatory assignment bug.
    The assignment requires little-endian byte order.
    Inputting 'AQI=' (Bytes [1, 2] in little-endian = 513) should return 513.
    """
    payload = {"input": "AQI=", "inputType": "base64", "outputType": "decimal"}
    response = client.post('/convert', json=payload)
    data = json.loads(response.data)
    
    # This test will FAIL on the buggy base repository code because it evaluates to 258 (big-endian)
    assert response.status_code == 200
    assert data['result'] == "513" 

def test_text_to_number_limited_dictionary(client):
    """Tests text_to_number logic limits (e.g., handles 'eleven' gracefully or fails)."""
    payload = {"input": "eleven", "inputType": "text", "outputType": "decimal"}
    response = client.post('/convert', json=payload)
    data = json.loads(response.data)
    
    assert response.status_code == 200
    assert data['result'] is None
    assert "Unable to convert text to number" in data['error']

def test_invalid_input_type(client):
    """Tests application error handling for invalid configuration options."""
    payload = {"input": "10", "inputType": "invalid_type", "outputType": "decimal"}
    response = client.post('/convert', json=payload)
    data = json.loads(response.data)
    
    assert response.status_code == 200
    assert data['result'] is None
    assert "Invalid input type" in data['error']

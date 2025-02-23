# SellEaseAI
SellEase AI is an AI-powered chatbot designed to enhance customer experience on e-commerce platforms.
import openai
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

# OpenAI API Key (Replace with your own key)
OPENAI_API_KEY = "your_openai_api_key"
SHOPIFY_API_KEY = "your_shopify_api_key"
SHOPIFY_STORE_URL = "your_shopify_store_url"
TWILIO_ACCOUNT_SID = "your_twilio_account_sid"
TWILIO_AUTH_TOKEN = "your_twilio_auth_token"
WHATSAPP_NUMBER = "your_whatsapp_number"

# Function to interact with OpenAI GPT
def get_ai_response(user_input):
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You are an AI chatbot for an e-commerce store, assisting customers with order tracking, product recommendations, and FAQs."},
            {"role": "user", "content": user_input}
        ]
    )
    return response["choices"][0]["message"]["content"].strip()

# Function to get order status from Shopify
def get_order_status(order_id):
    url = f"{SHOPIFY_STORE_URL}/admin/api/2023-01/orders/{order_id}.json"
    headers = {"X-Shopify-Access-Token": SHOPIFY_API_KEY}
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        return response.json()
    return {"error": "Unable to fetch order details"}

# Function to send WhatsApp message via Twilio
def send_whatsapp_message(to, message):
    url = f"https://api.twilio.com/2010-04-01/Accounts/{TWILIO_ACCOUNT_SID}/Messages.json"
    data = {
        "From": f"whatsapp:{WHATSAPP_NUMBER}",
        "To": f"whatsapp:{to}",
        "Body": message
    }
    auth = (TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN)
    response = requests.post(url, data=data, auth=auth)
    return response.json()

# API endpoint to receive user queries
@app.route("/chatbot", methods=["POST"])
def chatbot():
    user_message = request.json.get("message")
    if not user_message:
        return jsonify({"error": "No message provided"}), 400
    
    bot_response = get_ai_response(user_message)
    return jsonify({"response": bot_response})

# API endpoint for order status queries
@app.route("/order_status", methods=["POST"])
def order_status():
    order_id = request.json.get("order_id")
    if not order_id:
        return jsonify({"error": "No order ID provided"}), 400
    
    order_info = get_order_status(order_id)
    return jsonify(order_info)

# API endpoint to send WhatsApp messages
@app.route("/send_whatsapp", methods=["POST"])
def send_whatsapp():
    to = request.json.get("to")
    message = request.json.get("message")
    if not to or not message:
        return jsonify({"error": "Phone number and message required"}), 400
    
    response = send_whatsapp_message(to, message)
    return jsonify(response)

if __name__ == "__main__":
    app.run(debug=True)

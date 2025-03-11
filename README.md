Setup and Requirements
!pip install transformers torch textblob gradio

Import Required Libraries
from transformers import pipeline
from textblob import TextBlob
import random
import gradio as gr

Initialize the AI Model
generator = pipeline("text-generation", model="gpt2")

def generate_copy_hf(prompt, max_length=50):
    response = generator(prompt, max_length=max_length, num_return_sequences=1)
    return response[0]["generated_text"].strip()
    
Define the Marketing Copy Generator Function with Sentiment-Based Copywriting
def adjust_tone(prompt, tone):
    tone_prompts = {
        "Exciting": "Make this ad energetic and thrilling.",
        "Professional": "Make this ad sound formal and business-oriented.",
        "Casual": "Make this ad friendly and conversational."
    }
    return f"{tone_prompts.get(tone, '')} {prompt}"

def generate_marketing_copy(brand, product, audience, tone="Exciting"):
    prompt = (f"Write an ad for {brand}. The product is {product}. "
              f"The target audience includes {audience}. Provide a short, catchy headline followed by a marketing description.")
    
    adjusted_prompt = adjust_tone(prompt, tone)
    generated_text = generate_copy_hf(adjusted_prompt, max_length=80)
    
    # Splitting the response into headline and description
    sentences = generated_text.split(". ")
    headline = sentences[0] if sentences else "Experience Something Amazing!"
    description = " ".join(sentences[1:]) if len(sentences) > 1 else "Discover the features and benefits today!"
    
    return headline, description
    
Generate Hashtags and CTA Suggestions
def generate_hashtags(product):
    words = product.lower().split()
    hashtags = ["#" + word.capitalize() for word in words[:3]]  # Limit to 3 hashtags
    additional_tags = ["#Trending", "#MustHave", "#BestChoice"]
    return " ".join(hashtags + random.sample(additional_tags, 1))

def generate_cta():
    cta_options = [
        "Get yours today and stand out!",
        "Order now for an exclusive deal!",
        "Unlock the future – shop now!",
        "Act fast – limited stock available!",
        "Join thousands of happy customers!"
    ]
    return random.choice(cta_options)
    
Create Gradio Interface
def generate_ad_copy(brand, product, audience, tone):
    headline, description = generate_marketing_copy(brand, product, audience, tone)
    hashtags = generate_hashtags(product)
    cta = generate_cta()
    return headline, description, hashtags, cta

demo = gr.Interface(
    fn=generate_ad_copy,
    inputs=[
        gr.Textbox(label="Brand Name"),
        gr.Textbox(label="Product/Service Description"),
        gr.Textbox(label="Target Audience"),
        gr.Radio(["Exciting", "Professional", "Casual"], label="Tone", value="Exciting")
    ],
    outputs=[
        gr.Textbox(label="Ad Headline"),
        gr.Textbox(label="Marketing Description"),
        gr.Textbox(label="Hashtags"),
        gr.Textbox(label="Call to Action")
    ],
    title="AI-Powered Marketing Copy Generator",
    description="Generate highly engaging, brand-appropriate marketing copy with AI. Choose a tone, get compelling ad text, relevant hashtags, and strong CTAs."
)

demo.launch()

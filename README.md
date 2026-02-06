# MindLog-Core"""
Project: MindLog Core (v0.1.0)
Description: AI-Powered Patient Reported Outcome Engine.
             Converts unstructured patient narratives into structured clinical data.
Author: [kurotobi-hub]
License: MIT
"""import json
from openai import OpenAI

# Replace with your actual API Key
client = OpenAI(api_key="YOUR_API_KEY")

def analyze_patient_log(patient_text):
    """
    Analyzes raw patient diary text and converts it into structured clinical data.
    """
    
    # The "Brain" of the operation. 
    # Instructing the AI to think like an Occupational Therapist.
    system_prompt = """
    You are an expert Occupational Therapy (OT) assistant AI.
    Analyze the patient's diary entry and output the result ONLY in JSON format.
    Do not include any conversational text or markdown formatting outside the JSON.

    Output Schema:
    1. sentiment_score: Float between -1.0 (Despair) and 1.0 (Hope).
    2. activity_level: Integer from 1 (Bedridden) to 5 (Full Social Participation), inferred from context.
    3. pro_summary: A concise summary (max 20 words) of the Patient-Reported Outcome in English.
    4. risk_flag: Boolean (true/false). Set to true if there are signs of self-harm or critical deterioration.
    """

    try:
        response = client.chat.completions.create(
            model="gpt-4o", 
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": f"Patient Log: {patient_text}"}
            ],
            response_format={"type": "json_object"} # Enforce valid JSON structure
        )
        return json.loads(response.choices[0].message.content)
    except Exception as e:
        return {"error": str(e)}

# --- Simulation / Test Run ---

# Case 1: A Good Day (High engagement)
log_good = "I finally walked to the garden today. The breeze felt amazing. My knee hurts a bit, but it feels so much better than the gloom I felt yesterday."

# Case 2: A Bad Day (Low engagement, depressive symptoms)
log_bad = "I don't feel like doing anything anymore. Rehab feels pointless. I just want to stay in bed all day."

print("--- Case 1: Good Day ---")
print(json.dumps(analyze_patient_log(log_good), indent=2))

print("\n--- Case 2: Bad Day ---")
print(json.dumps(analyze_patient_log(log_bad), indent=2))



---

# **Centria FAQ Chatbot (AWS Lambda + Gradio + Groq)**
> A **serverless chatbot** using **AWS Lambda, Gradio, and Groq** to answer FAQs about Centria University.  
> - 📌 **FAQs** are prioritized.  
> - 📌 **Groq provides general answers** about Centria if no FAQ is found.  
> - 📌 **Handles greetings politely.**  

---

## **🌟 Features**
✅ **Serverless architecture** using AWS Lambda.  
✅ **Automatic responses** for FAQs using fuzzy matching.  
✅ **Groq AI-powered answers** if no FAQ match is found.  
✅ **Secure API integration** using AWS IAM permissions.  
✅ **Easy-to-use Chat Interface** with Gradio.

---

## **🛠️ Tech Stack**
- **AWS Lambda** - Serverless backend.  
- **Groq API** - AI-powered answers for general queries.  
- **Gradio** - Interactive web interface.  
- **IAM (AWS Identity & Access Management)** - Secure API permissions.  
- **Python** - Backend implementation.  

---

# **🚀 Deployment Steps**
## **1️⃣ Set Up AWS Lambda**
AWS Lambda runs the chatbot logic. Follow these steps to deploy:

### **🔹 Step 1: Create a New Lambda Function**
1. Go to **AWS Lambda Console** → Click `Create function`.
2. Choose:
   - **Function Name:** `centria-faq-bot`
   - **Runtime:** `Python 3.9`
   - **Execution Role:** Select **“Create a new role with basic permissions”**.
3. Click `Create Function`.

📌 **Refer to images:**  
📷 `lambda-one.png` | 📷 `lambda-two` | 📷 `lambda-three.png` | 📷 `lambda-four.png` | 📷 `lambda-five.png`

### **🔹 Step 2: Add Your Lambda Code**
- Open the **AWS Lambda Console**, select your function.
- Click `Edit` → Paste the following **Lambda function code**:

```python
import os
import json
import asyncio
import difflib
from groq import AsyncGroq

# Retrieve environment variables
GROQ_API_KEY = os.environ.get("GROQ_API_KEY")
GROQ_MODEL_ID = os.environ.get("GROQ_MODEL_ID")

# Initialize Groq client
groq_client = AsyncGroq(api_key=GROQ_API_KEY)

# ✅ FAQs Dictionary
FAQs = {
    "What is the exam structure, grading and testing and process of examinations like? What is the scoring process? How much of a grade is based on exams or projects?":
        "The structure of the course exams depends on the subject and the course. Sometimes the exam contains essays, sometimes multiple-choice tasks. Quite often there is no exam at all, and instead, there is project work, teamwork, or similar activities. Evaluation is usually on a scale of 0–5, sometimes pass–failed. The teacher is responsible for the assessment and announces the scoring process and evaluation at the beginning of the course.",
    
    "Can we complete our studies or single courses faster if we want? How and how fast?":
        "It is possible. The completion time depends on the student’s background, level of competence, and willingness to make an effort. However, the degree is structured to take 3.5 years, and students are encouraged not to rush the courses to fully absorb the content.",
    
    "How much time is there for research during studies?":
        "There are special courses for research work in the BBA curriculum, such as 'Research work' and 'Thesis process.' Additionally, smaller research projects are included in the study programme, allowing students to participate in R&D projects in collaboration with businesses.",
    
    "Will we be provided our own laptops or must we buy our own?":
        "Students must have their own laptops. However, computers and printers are available on Centria campuses and in city libraries for students who do not have a laptop initially.",
    
    "I would like to know more about the teaching methods and evaluation.":
        "The teaching methods vary. In daytime groups, courses are mainly on-campus, with classroom lectures. Some online courses are available. In blended learning groups, lectures are primarily online, with one in-person day on campus in Pietarsaari every other week.",
    
    "What is the curriculum like?":
        "You can view Centria’s Study Guide here: [Study Guide](https://opinto-opas.fi). More information is also available here: [Content of Studies – Centria](https://net.centria.fi/en/education/faq/).",
    
    "What is the average class size?":
        "Usually, there are 25–30 students in one group, but the number of students varies.",
    
    "Does Centria offer Finnish language courses?":
        "Yes, 9 ECTS of Finnish language courses are compulsory for BBA students. Optional courses, such as 'Finnish for Working Life' and 'Spoken Finnish,' are highly recommended to improve employability in Finland. Up to 30 ECTS of Finnish courses are available.",
    
    "How is the city life in Kokkola and Pietarsaari?":
        "Kokkola describes itself as 'a seaside town with an inspiring atmosphere, community spirit, and a good everyday life.' More details: [Visit Kokkola](https://www.visitkokkola.fi) and [Info Finland](https://www.infofinland.fi). Pietarsaari is a lively town with diverse sports facilities, a long history, and good job opportunities. Learn more: [Visit Pietarsaari](https://www.visitpietarsaarenseutu.fi).",
    
    "How is life in Finland in general?":
        "Finland is considered the happiest country in the world. It is known for nature, silence, great education, calm people, and distinct seasons. Explore more: [Visit Finland](https://www.visitfinland.com) and [Info Finland](https://www.infofinland.fi).",
    
    "How do you deal with the different seasons and especially the dark winter days?":
        "Winter can be challenging. It’s important to exercise, spend time outdoors, take vitamin D, and engage in activities like skiing and watching the northern lights.",
    
    "How do you cope with the language and the people?":
        "Finnish and Swedish may seem difficult, but language learning resources like Duolingo can help. It is advisable to study Finnish outside of courses to improve job prospects.",
    
    "Are there clubs and organizations for extracurricular activities?":
        "Student Union COPSA organizes activities and events for students. Follow them on [Facebook](https://www.facebook.com/OpiskelijakuntaCOPSA) and [Instagram](https://www.instagram.com/opiskelijakuntacopsa).",
    
    "What is student housing like in Kokkola and Pietarsaari?":
        "In Kokkola, Tankkari Student Housing offers shared apartments and studios. In Pietarsaari, Fastighets Ab Ebba Kiinteistö Oy provides student apartments. More details: [Tankkari](https://www.tankkari.fi) and [Ebba Kiinteistö](https://www.ebbafastigheter.fi).",
    
    "How much do I need to cover my accommodation per month, except for tuition fee?":
        "Student apartments typically cost 300–500 EUR per month, while private market rentals range from 500–800 EUR. Monthly living expenses are estimated at 700–900 EUR, depending on personal spending habits.",
    
    "Do you have university accommodation for families?":
        "In Kokkola, Tankkari Student Housing has a limited number of family apartments. In Pietarsaari, family apartments can be applied for through [Ebba Kiinteistö](https://www.ebbafastigheter.fi).",
    
    "How do I get a student apartment in Kokkola?":
        "Apply through [Kt Oy Tankkari](https://www.tankkari.fi) at least 2–3 months before arrival. If apartments are unavailable, check Kokkola City Rental Apartments or [Vuokraovi.com](https://www.vuokraovi.com).",
    
    "What do I need to take into consideration when applying for a residence permit to study in Finland?":
        "Refer to [Migri](https://www.migri.fi) for detailed information on residence permit requirements for students.",
    
    "Can I bring my family with me?":
        "Yes. More details are available at [Moving to Finland to be with a family member](https://www.migri.fi).",
    
    "What happens if I fail in a visa interview?":
        "Contact Finnish Immigration Services at [Migri](https://www.migri.fi). If you receive a negative decision, inform Centria’s Admissions Service at [admissions@centria.fi](mailto:admissions@centria.fi).",
    
    "What are the regulations with working while studying?":
        "Students can work unlimited hours if the job is part of practical training or diploma work. Otherwise, the maximum is 30 hours per week, averaged over a year.",
    
    "Does the school provide airport pickup?":
        "No, Centria does not provide airport pickup. Refer to [New Students – Centria](https://net.centria.fi/en) for travel details.",
    
    "Will there be practical training positions or assistance from school faculty members?":
        "Centria UAS offers Career and Work-Life Services to help students find practical training positions. More details: [Career and Work-Life Services](https://net.centria.fi/en/for-students/students-guide/career-and-work-life-services/).",
    
    "What are the practical training and networking options available?":
        "Centria organizes recruitment events where students can meet company representatives. Career and Work-Life Services assist in finding internships and jobs.",
    
    "Are there enough IT companies in Kokkola for a possible practical training period?":
        "Yes, several IT companies operate in Kokkola. Centria partners with many of them to provide training opportunities.",
    
    "What professional opportunities does Kokkola offer for Environmental Chemistry students?":
        "Kokkola has major companies offering summer and post-graduation jobs in the chemistry sector. Over 1,500 new workers are expected to be needed in coming years.",
    
    "What kinds of professions do students from different programs go into?":
        "Career and Work-Life Services help students find paths to employment based on their study field.",
    
    "How difficult is it for students to get jobs during breaks?":
        "There are part-time job opportunities in the region. Students who actively seek jobs and network typically find employment.",
    
    "What are the possibilities of getting a job without knowing Finnish?":
        "While possible, knowing Finnish significantly improves job prospects. It is recommended to start learning Finnish early.",
    
    "Useful websites for finding work and practical training positions?":
        "Refer to: [Career and Work-Life Services](https://net.centria.fi/en/for-students/students-guide/career-and-work-life-services/), [Job Market](https://tyomarkkinatori.fi/en), and [Work in Finland](https://www.workinfinland.com/en/open-jobs/)."
}

# ✅ Common Greetings
GREETINGS = ["hello", "hi", "hey", "good morning", "good afternoon", "good evening"]

# ✅ Function to find the closest FAQ match
def find_best_faq_match(user_query):
    """Finds the closest FAQ match using fuzzy matching."""
    user_query = user_query.lower()
    faq_keys = list(FAQs.keys())
    closest_matches = difflib.get_close_matches(user_query, faq_keys, n=1, cutoff=0.7)
    return closest_matches[0] if closest_matches else None

# ✅ Function to Generate Response from Groq
async def generate_groq_response(prompt, user_query):
    """Generates a refined response using the Groq API."""
    try:
        response = await groq_client.chat.completions.create(
            messages=[
                {
                    "role": "system",
                    "content": (
                        "You are a factual assistant for Centria University. First, check the provided FAQs and prioritize accurate information. "
                        "If no match is found in the FAQs, provide general knowledge about Centria University based on trusted sources."
                        "If the user greets you, respond politely."
                    ),
                },
                {
                    "role": "user",
                    "content": f"The user asked: '{user_query}'. Here are some FAQs: {json.dumps(FAQs)}. Check if the question matches any FAQ first, then answer.",
                }
            ],
            model=GROQ_MODEL_ID,
            max_tokens=300,
            temperature=0.3,
            top_p=0.8
        )
        return response
    except Exception as e:
        print(f"Error invoking Groq: {e}")
        return None

# ✅ Lambda Handler Function
def lambda_handler(event, context):
    """Handles incoming API Gateway requests."""
    print("Received event:", event)

    try:
        # ✅ Fix: Ensure Proper JSON Parsing
        try:
            body = json.loads(event["body"]) if "body" in event else event
            user_query = body.get("query", "").strip().lower()
        except json.JSONDecodeError:
            return {
                "statusCode": 400,
                "headers": {"Content-Type": "application/json"},
                "body": json.dumps({"error": "Invalid JSON format."})
            }

        if not user_query:
            return {
                "statusCode": 400,
                "headers": {"Content-Type": "application/json"},
                "body": json.dumps({"error": "No query provided."})
            }

        # ✅ **1️⃣ Handle Greetings Politely**
        if user_query in GREETINGS:
            return {
                "statusCode": 200,
                "headers": {"Content-Type": "application/json"},
                "body": json.dumps({"answer": "Hello! How can I assist you with questions about Centria University?"})
            }

        # ✅ **2️⃣ Check FAQs for a Direct Answer**
        best_faq_match = find_best_faq_match(user_query)
        if best_faq_match:
            return {
                "statusCode": 200,
                "headers": {"Content-Type": "application/json"},
                "body": json.dumps({"answer": FAQs[best_faq_match]})
            }

        # ✅ **3️⃣ Use Groq as the Knowledge Source**
        prompt = "The user is asking about Centria University. Please provide an accurate response."
        
        try:
            loop = asyncio.get_event_loop()
        except RuntimeError:
            loop = asyncio.new_event_loop()
            asyncio.set_event_loop(loop)

        try:
            groq_response = loop.run_until_complete(generate_groq_response(prompt, user_query))
        except Exception as e:
            print("Error running Groq async call:", str(e))
            groq_response = None

        # ✅ **4️⃣ Extract response from Groq**
        try:
            if groq_response and hasattr(groq_response, "choices") and len(groq_response.choices) > 0:
                answer = groq_response.choices[0].message.content
            else:
                answer = "No relevant information found."
        except Exception as e:
            print("Error extracting answer from Groq response:", str(e))
            answer = "Error generating answer."

        return {
            "statusCode": 200,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"answer": answer})
        }

    except Exception as e:
        print("Lambda Error:", str(e))
        return {
            "statusCode": 500,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"error": "Internal server error."})
        }

```

4. Click **Deploy**.

---

## **2️⃣ Set Up AWS IAM (Permissions)**
AWS IAM allows secure access control.

### **🔹 Step 1: Create an IAM Role**
1. Go to **AWS IAM Console** → Click `Roles` → `Create Role`.
2. Choose **Lambda** as the use case.
3. Attach **AWSLambdaBasicExecutionRole** policy.
4. Click **Next**, name the role: `centria-lambda-role`, and click `Create role`.

📌 **Refer to images:**  
📷 `IAM-one.png` | 📷 `IAM-two.png` | 📷 `IAM-three.png` | 📷 `IAM-four.png`

---

## **3️⃣ Configure AWS API Gateway**
To make the Lambda function accessible via **Gradio**, set up API Gateway.

1. Go to **AWS API Gateway** → Click `Create API`.
2. Select **HTTP API** → `Add Integration` → Choose **AWS Lambda**.
3. Select your function `centria-faq-bot` and click **Create**.
4. Deploy the API and copy the **API Endpoint URL**.

---

## **4️⃣ Deploy Gradio Chat Interface**
1. Install **Gradio** and **Requests**:
   ```bash
   pip install gradio requests
   ```
2. Create a Python file `app.py` and paste the following:

```python
import gradio as gr
import requests
import json

# AWS Lambda Function URL
FUNCTION_URL = "https://a2s74dy7a2qwuyxoyq4o47u3ci0kxhto.lambda-url.eu-west-1.on.aws/"

def chat_with_nri(message, history):
    """
    Sends user message to AWS Lambda and returns the bot response.
    Ensures history is formatted correctly for Gradio.
    """

    # Prepare the JSON payload
    payload = json.dumps({"query": message})

    try:
        response = requests.post(FUNCTION_URL, data=payload, headers={"Content-Type": "application/json"})
        print("Lambda status code:", response.status_code)
        print("Lambda response text:", response.text)

        if response.status_code == 200:
            result = response.json()
            answer = result.get("answer", "No answer returned.")
        else:
            answer = f"Error: {response.status_code} - {response.text}"
    except Exception as e:
        answer = f"Exception: {str(e)}"

    # ✅ Fix: Return a dictionary with the expected structure
    return answer


# ✅ Fix: Use gr.ChatInterface() correctly
with gr.Blocks() as demo:
    gr.ChatInterface(
        fn=chat_with_nri,
        title="Centria FAQ Chatbot",
        description="Chat with the FAQ chatbot powered by AWS Lambda."
    )

demo.launch(share=True)

```

3. Run the chatbot:
   ```bash
   python app.py
   ```

4. Open the **Gradio link** generated in the terminal.

---

## **5️⃣ Configure Groq API**
1. Sign up at [Groq API](https://groq.com).
2. Generate an API key.
3. Add your API Key to **AWS Lambda** Environment Variables:
   - Key: `GROQ_API_KEY`
   - Value: `<YOUR_GROQ_API_KEY>`

---

## **6️⃣ Configure AWS Kendra (Optional)**
📌 *Skip this if only using FAQs + Groq.*  
1. Open **AWS Kendra Console** → Click `Create an Index`.
2. Add **Centria University data sources**.
3. Deploy and copy the **Kendra Index ID**.

📌 **Refer to images:**  
📷 `kendra-one.png` | 📷 `kendra-two.png` | 📷 `kendra-three.png`

---

# **💻 Running the Chatbot Locally**
1. Clone this repo:
   ```bash
   git clone https://github.com/your-username/centria-chatbot.git
   cd centria-chatbot
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Run the chatbot:
   ```bash
   python app.py
   ```
4. Open **Gradio UI** in the browser.

---

# **🔧 Troubleshooting**
### **Issue: Lambda not returning a response?**
- Check **CloudWatch Logs** for errors.
- Ensure `API Gateway` is correctly linked.

### **Issue: API not responding?**
- Verify API Gateway **Deployment Stage**.
- Check **IAM Role permissions**.

### **Issue: Gradio UI not loading?**
- Run `pip install gradio requests` and restart.

---

# **🌟 Future Enhancements**
✅ **Multi-turn conversations** for better user experience.  
✅ **Database logging** for analytics.  
✅ **Voice-to-text support** for accessibility.  

---

# **📜 License**
MIT License. Feel free to modify and use.  

---

# **🚀 Contributors**
- **[Nofiu Moruf Pelumi]** - Developer  
- **[Ojugbele Daniel Babatunde]** - Support  

---

# **📞 Contact**
For any issues, reach out via:  
📧 Email: `nofiumoruf17@gmail.com`  
🔗 GitHub Issues: [Report Issue](https://github.com/nofiupelumi/AWS-CHATBOT/issues)

---


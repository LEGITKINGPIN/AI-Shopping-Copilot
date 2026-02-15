# 🛍️ AI Shopping Copilot for Retail

### *A Trust-Driven Decision Intelligence Layer for the "AI for Bharat" Hackathon*

> **The Problem:** Indian e-commerce is broken. 60% of reviews are fake, "sales" are often deceptive price hikes, and fashion return rates hit 40% due to size mismatches.
> **The Solution:** We are not building a chatbot. We are building a **Decision Intelligence Layer** that actively guides, verifies, and protects the consumer using Generative AI and Deterministic Algorithms.

---

## 🚀 Key Features (The 3 Pillars)

### 🧠 1. Decision Intelligence (The "Brain")

* **Context-Aware AI Salesman:** Instead of 10,000 search results, get 3 intelligent recommendations based on your specific needs (e.g., *"Find me a gaming phone under 20k with liquid cooling"*).
* **Persistent Memory:** The AI remembers your preferences (e.g., *"I prefer battery life over camera"*) across sessions.

### 🛡️ 2. The Trust Engine (The "Verifier")

* **Fake Review Detection:** A deterministic algorithm that flags products with suspicious review patterns (bot comments, temporal bursts).
* **Deceptive Pricing Alert:** Tracks historical pricing to expose "Fake Sales" where the MRP was inflated just before a discount.
* **Trust Score (0-100):** A unified metric displayed on every product card.

### 📉 3. Risk Reduction (The "Protector")

* **Return Risk Predictor:** Calculates the probability (%) of a return before you buy.
* **Smart Fit Technology:** Maps your body measurements to the product's size chart to warn you about fit issues (e.g., *"⚠️ High Risk: This brand runs small. Size M will be tight on your 40-inch chest."*).

---

## 🛠️ Tech Stack

| Component | Technology | Description |
| --- | --- | --- |
| **GenAI Core** | **Amazon Bedrock (Claude 3 Sonnet)** | Natural language understanding and intent detection. |
| **Compute** | **AWS Lambda (Python)** | Serverless execution of Trust & Risk algorithms. |
| **Database** | **Amazon DynamoDB** | Single-table design for millisecond-latency access to user profiles & product data. |
| **Frontend** | **Streamlit** | Interactive "Wizard of Oz" dashboard for the hackathon demo. |
| **Data Mocking** | **Faker / Pandas** | Synthetic dataset generation simulating Indian retail scenarios. |

---

## 📂 Repository Structure

```text
/ai-shopping-copilot
│
├── /src
│   ├── app.py                 # Main Streamlit Application (Frontend)
│   ├── trust_engine.py        # Algorithm for Trust Score (Fake Reviews/Price)
│   ├── risk_engine.py         # Algorithm for Return Risk (Smart Fit)
│   ├── data_generator.py      # Script to generate synthetic "Bad Actor" data
│   └── bedrock_agent.py       # (Simulation) Conversational Logic
│
├── /data
│   └── mock_database.json     # The "Source of Truth" (Users & Products)
│
├── requirements.txt           # Python Dependencies
├── README.md                  # Project Documentation
└── architecture_diagram.png   # System Architecture Visual

```

---

## ⚙️ Installation & Setup

1. **Clone the Repository**
```bash
git clone https://github.com/your-username/ai-shopping-copilot.git
cd ai-shopping-copilot

```


2. **Install Dependencies**
```bash
pip install -r requirements.txt

```


3. **Generate Data (Crucial Step)**
* This script creates the synthetic dataset with injected "Bad Actors" (Fake reviews, Price hikes) for the demo.


```bash
python src/data_generator.py

```


4. **Run the Application**
```bash
streamlit run src/app.py

```



---

## 📸 Demo Scenarios (How to Test)

Use the predefined users in the sidebar to see different outcomes:

1. **Scenario A: The "Trust" Check**
* Search for **"Generic Smartwatch"**.
* **Observation:** Notice the **Red Shield Icon**. The system detects 45 duplicate reviews and flags it as "Low Trust".


2. **Scenario B: The "Fake Sale" Check**
* Search for **"Gaming Monitor"**.
* **Observation:** The system flags a "Deceptive Pricing Alert" because the price was hiked by 50% yesterday.


3. **Scenario C: The "Smart Fit" Check**
* Select User: **Rahul (Chest: 40 inches)**.
* View Product: **"Slim Fit Kurta (Size M)"**.
* **Observation:** A **High Risk Warning (85%)** pops up because Size M is 38 inches (Too tight).



---

## 🔮 Future Roadmap

* [ ] **Computer Vision:** Allow users to upload photos for auto-measurement (using Amazon Titan Multimodal).
* [ ] **Real-Time Integration:** Connect to live Flipkart/Amazon APIs for real-world scraping.
* [ ] **Voice Interface:** Enable voice-based shopping using Amazon Transcribe.

---

## 👥 Collaborators
* **Meet**
* **Vedant**
* **Ananya**
* **Mahima**

---

### 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://www.google.com/search?q=LICENSE) file for details.
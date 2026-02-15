# Design Document: AI Shopping Copilot for Retail

## Overview

The AI Shopping Copilot is a Python-based trust intelligence system for Indian e-commerce that combines conversational product discovery, algorithmic trust scoring, and predictive return risk analysis. The system operates as a standalone Streamlit application with three core subsystems:

1. **Conversational Shopping Assistant**: Multi-turn dialogue system that clarifies user needs and recommends products
2. **Trust Engine**: Algorithmic scoring system that detects fake reviews and deceptive pricing
3. **Return Risk Predictor**: Machine learning system that predicts return probability using fit analysis and behavioral patterns

The architecture prioritizes simplicity and demonstrability, using synthetic data generation to eliminate external dependencies while showcasing real-world problem-solving capabilities.

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Streamlit UI Layer                       │
│  (Chat Interface, Product Cards, Trust Badges, Modals)      │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────┴───────────────────────────────────────┐
│                  Application Controller                      │
│         (Session Management, Flow Orchestration)             │
└─────┬──────────────┬──────────────┬─────────────────────────┘
      │              │              │
┌─────▼─────┐  ┌────▼─────┐  ┌────▼──────────┐
│ Shopping  │  │  Trust   │  │ Return Risk   │
│ Copilot   │  │  Engine  │  │  Predictor    │
└─────┬─────┘  └────┬─────┘  └────┬──────────┘
      │              │              │
      │         ┌────▼──────────────▼────┐
      │         │  Smart Fit Intelligence │
      │         └─────────────────────────┘
      │
┌─────▼──────────────────────────────────────┐
│         Demo Data Generator                 │
│  (Users, Products, Reviews, Price History) │
└─────────────────────────────────────────────┘
```

### Data Flow

1. **Initialization**: Demo Data Generator creates synthetic users, products, reviews, and price history
2. **Conversation**: User interacts with Shopping Copilot through Streamlit chat interface
3. **Recommendation**: Copilot queries product database and returns filtered recommendations
4. **Trust Analysis**: Trust Engine calculates Review Trust Score and Deal Authenticity for each product
5. **Risk Assessment**: When user views apparel, Return Risk Predictor and Smart Fit Intelligence analyze fit and return probability
6. **Display**: Streamlit UI renders products with trust badges, pricing warnings, and risk indicators

### Technology Stack

- **Frontend**: Streamlit (Python web framework)
- **Backend Logic**: Pure Python with NumPy for numerical operations
- **Data Generation**: Faker library for synthetic data
- **Sentiment Analysis**: TextBlob or VADER for review sentiment
- **ML**: Scikit-learn for logistic regression (return prediction)
- **Data Storage**: In-memory Python dictionaries and Pandas DataFrames

## Components and Interfaces

### 1. Demo Data Generator

**Purpose**: Generate realistic synthetic data for demonstration without external dependencies.

**Data Structures**:

```python
User = {
    'user_id': str,
    'name': str,
    'location': str,
    'measurements': {
        'height_cm': float,
        'weight_kg': float,
        'chest_cm': float,
        'waist_cm': float,
        'hips_cm': float
    },
    'behavioral_profile': {
        'total_orders': int,
        'total_returns': int,
        'return_rate': float,  # 0.0 to 1.0
        'bracketing_tendency': float  # 0.0 to 1.0
    },
    'preferences': {
        'preferred_brands': List[str],
        'deal_sensitivity': float  # 0.0 to 1.0
    }
}

Product = {
    'product_id': str,
    'category': str,  # 'Electronics' or 'Fashion'
    'subcategory': str,
    'brand': str,
    'title': str,
    'image_url': str,  # placeholder URL
    'pricing': {
        'mrp': float,
        'current_price': float,
        'price_history': List[{'date': str, 'price': float}]  # 30 days
    },
    'reviews': List[Review],
    'sizing_data': Optional[SizingData]  # Only for Fashion
}

Review = {
    'review_id': str,
    'reviewer_id': str,
    'rating': int,  # 1-5
    'text': str,
    'is_verified': bool,
    'date': str,
    'account_age_days': int
}

SizingData = {
    'size_chart': {
        'S': {'chest': float, 'waist': float, 'hips': float},
        'M': {'chest': float, 'waist': float, 'hips': float},
        'L': {'chest': float, 'waist': float, 'hips': float},
        'XL': {'chest': float, 'waist': float, 'hips': float}
    },
    'fit_feedback': List[str]  # ["runs small", "true to size", etc.]
}
```

**Generation Strategy**:

- **Users**: Use Faker to generate 50 Indian names and cities. Randomly assign measurements following normal distributions (e.g., height: mean=165cm, std=10cm). Assign behavioral profiles with varying return rates (10-50%) and bracketing tendencies.

- **Products**: Create 200 products (120 Electronics, 80 Fashion). For each product:
  - Generate realistic titles and brands
  - Create 30-day price history with some products showing MRP inflation before "sales"
  - Generate 5-20 reviews per product with varying patterns

- **Bad Actor Injection**:
  - 20 products with fake reviews: Generic 5-star text ("Great product!", "Excellent!"), posted in temporal bursts (multiple reviews same day), low verified purchase ratio
  - 15 products with deceptive pricing: MRP increased 20-30% in days 25-30, then "discounted" to near-original price
  - 25 apparel products with poor fit: Size charts that don't match standard measurements, negative fit feedback in reviews

**Interface**:

```python
class DemoDataGenerator:
    def generate_users(count: int) -> List[User]
    def generate_products(count: int, bad_actor_config: dict) -> List[Product]
    def generate_reviews(product: Product, pattern: str) -> List[Review]
    def generate_price_history(base_price: float, pattern: str) -> List[dict]
```

### 2. Shopping Copilot

**Purpose**: Conversational agent that clarifies user needs and recommends products.

**Conversation State**:

```python
ConversationState = {
    'user_id': str,
    'turn_count': int,
    'collected_criteria': {
        'category': Optional[str],
        'budget_min': Optional[float],
        'budget_max': Optional[float],
        'use_case': Optional[str],
        'brand_preference': Optional[str],
        'other_preferences': List[str]
    },
    'history': List[{'role': str, 'content': str}]
}
```

**Logic Flow**:

1. **Initial Turn**: Greet user and ask about shopping needs
2. **Clarification Turns**: If criteria incomplete, ask targeted questions:
   - If no category: "What are you looking for? Electronics or Fashion?"
   - If no budget: "What's your budget range?"
   - If category is Fashion and no use-case: "What's the occasion? Casual, formal, or sportswear?"
3. **Recommendation Turn**: When sufficient criteria collected (category + budget), query product database and return top 3 matches
4. **Filtering Logic**: 
   - Filter by category
   - Filter by price range (budget_min <= current_price <= budget_max)
   - Sort by Review Trust Score (descending)
   - Return top 3

**Interface**:

```python
class ShoppingCopilot:
    def __init__(self, products: List[Product])
    def process_user_message(self, message: str, state: ConversationState) -> str
    def extract_criteria(self, message: str, state: ConversationState) -> None
    def is_ready_to_recommend(self, state: ConversationState) -> bool
    def generate_recommendations(self, state: ConversationState) -> List[Product]
```

### 3. Trust Engine

**Purpose**: Calculate Review Trust Score and detect pricing manipulation.

**Trust Score Calculation**:

The Review Trust Score (RTS) is computed using a weighted formula:

```
RTS = (W_sentiment × Sentiment_Score) + (W_verified × Verified_Ratio) + 
      (W_account × Account_Score) - (W_burst × Burst_Penalty) - 
      (W_pricing × Pricing_Penalty)

Where:
- W_sentiment = 0.35
- W_verified = 0.25
- W_account = 0.20
- W_burst = 0.15
- W_pricing = 0.15
```

**Component Calculations**:

1. **Sentiment_Score** (0-100):
   - For each review: Extract sentiment polarity using TextBlob (-1 to +1)
   - Normalize sentiment to 0-1 scale: normalized_sentiment = (polarity + 1) / 2
   - Normalize rating to 0-1 scale: normalized_rating = rating / 5
   - Calculate consistency: consistency = 1 - |normalized_sentiment - normalized_rating|
   - Aggregate: Sentiment_Score = (sum of all consistency scores / review count) × 100

2. **Verified_Ratio** (0-100):
   - Verified_Ratio = (count of verified reviews / total reviews) × 100

3. **Account_Score** (0-100):
   - For each review: account_credibility = min(account_age_days / 365, 1.0) × 100
   - Aggregate: Account_Score = average of all account_credibility scores

4. **Burst_Penalty** (0-100):
   - Group reviews by date
   - If any single day has > 30% of total reviews: Burst_Penalty = 50
   - If any 3-day window has > 60% of total reviews: Burst_Penalty = 30
   - Otherwise: Burst_Penalty = 0

5. **Pricing_Penalty** (0-100):
   - Calculate 30-day average price: avg_price = mean(price_history)
   - If MRP > avg_price × 1.15: Pricing_Penalty = 40
   - Otherwise: Pricing_Penalty = 0

**Final Score**: Clamp RTS to [0, 100] range.

**Deal Authenticity Analysis**:

```python
claimed_discount = ((mrp - current_price) / mrp) × 100
actual_discount = ((avg_30day_price - current_price) / avg_30day_price) × 100
is_authentic_deal = |claimed_discount - actual_discount| <= 10
```

**Interface**:

```python
class TrustEngine:
    def calculate_review_trust_score(self, product: Product) -> dict
    def analyze_sentiment_consistency(self, reviews: List[Review]) -> float
    def detect_burst_pattern(self, reviews: List[Review]) -> float
    def analyze_deal_authenticity(self, pricing: dict) -> dict
    def get_trust_badge(self, score: float) -> str  # 'green', 'amber', 'red'
```

### 4. Return Risk Predictor

**Purpose**: Predict return probability using fit analysis and behavioral patterns.

**Logistic Regression Model**:

```
logit = β0 + β1×bracketing + β2×user_return_rate + β3×category_risk + 
        β4×fit_delta + β5×sizing_feedback

return_probability = (1 / (1 + e^(-logit))) × 100
```

**Feature Engineering**:

1. **bracketing** (0 or 1): 1 if user has multiple sizes of same product in cart
2. **user_return_rate** (0-1): From user behavioral profile
3. **category_risk** (0-1): Baseline return rate for category (Fashion: 0.4, Electronics: 0.15)
4. **fit_delta** (0-1): Normalized total fit delta from Smart Fit Intelligence
5. **sizing_feedback** (-0.2 to +0.2): Adjustment based on review signals
   - "runs small": +0.2
   - "runs large": +0.2
   - "true to size": -0.2

**Coefficient Values** (tuned for demo):
- β0 = -2.0 (intercept)
- β1 = 1.5 (bracketing)
- β2 = 2.0 (user return rate)
- β3 = 1.0 (category risk)
- β4 = 1.8 (fit delta)
- β5 = 0.8 (sizing feedback)

**Interface**:

```python
class ReturnRiskPredictor:
    def predict_return_probability(self, user: User, product: Product, cart_context: dict) -> float
    def extract_features(self, user: User, product: Product, cart_context: dict) -> dict
    def calculate_logit(self, features: dict) -> float
```

### 5. Smart Fit Intelligence

**Purpose**: Recommend optimal size for apparel based on body measurements.

**Fit Delta Calculation**:

For each size in size chart:
```
chest_delta = |user.measurements.chest_cm - size_chart[size].chest|
waist_delta = |user.measurements.waist_cm - size_chart[size].waist|
hips_delta = |user.measurements.hips_cm - size_chart[size].hips|

total_fit_delta = chest_delta + waist_delta + hips_delta
```

**Size Recommendation Logic**:

1. Calculate total_fit_delta for all available sizes
2. Select size with minimum total_fit_delta
3. If minimum fit_delta > 15cm: Flag as "Poor Fit Risk"
4. If any single dimension delta > 5cm: Note specific fit issue
5. Adjust recommendation based on fit_feedback from reviews:
   - If "runs small" is dominant and recommended size is not largest: Suggest one size up
   - If "runs large" is dominant and recommended size is not smallest: Suggest one size down

**Interface**:

```python
class SmartFitIntelligence:
    def recommend_size(self, user: User, product: Product) -> dict
    def calculate_fit_delta(self, user_measurements: dict, size_spec: dict) -> float
    def analyze_fit_feedback(self, reviews: List[Review]) -> str  # 'runs_small', 'true_to_size', 'runs_large'
    def get_fit_explanation(self, user: User, recommended_size: str, product: Product) -> str
```

### 6. Streamlit UI Controller

**Purpose**: Orchestrate user interactions and render visual components.

**Session State Management**:

```python
st.session_state = {
    'conversation_state': ConversationState,
    'current_user': User,
    'products': List[Product],
    'selected_product': Optional[Product],
    'cart': List[dict],
    'initialized': bool
}
```

**UI Components**:

1. **Chat Interface**: 
   - Display conversation history
   - Text input for user messages
   - Auto-scroll to latest message

2. **Product Card**:
   - Product image (placeholder)
   - Title, brand, category
   - Price with strikethrough MRP
   - Trust badge (green/amber/red circle with score)
   - "View Details" button

3. **Product Detail View**:
   - Full product information
   - Trust Score gauge (0-100 with color gradient)
   - Trust score breakdown (sentiment, verified ratio, account score, penalties)
   - Pricing authenticity section (if deceptive: warning banner with actual vs claimed discount)
   - For apparel: Size recommendation with fit analysis
   - Reviews section
   - "Add to Cart" button

4. **Return Risk Modal**:
   - Triggered when adding high-risk product to cart
   - Display return probability percentage
   - Show contributing factors (fit issues, user history, bracketing)
   - Size recommendation if applicable
   - "Proceed Anyway" and "Cancel" buttons

**Interface**:

```python
class StreamlitUIController:
    def render_chat_interface(self)
    def render_product_card(self, product: Product, trust_data: dict)
    def render_product_detail(self, product: Product, trust_data: dict, risk_data: dict)
    def render_trust_gauge(self, score: float)
    def render_return_risk_modal(self, risk_data: dict)
    def handle_user_input(self, message: str)
    def handle_product_selection(self, product_id: str)
    def handle_add_to_cart(self, product_id: str)
```

## Data Models

### Product Database Schema

Products are stored as a list of dictionaries in memory, indexed by product_id for O(1) lookup.

```python
products_db: Dict[str, Product] = {
    'prod_001': {
        'product_id': 'prod_001',
        'category': 'Fashion',
        'subcategory': 'Shirts',
        'brand': 'FabIndia',
        'title': 'Cotton Casual Shirt',
        'image_url': 'https://via.placeholder.com/300',
        'pricing': {
            'mrp': 1999.0,
            'current_price': 1299.0,
            'price_history': [
                {'date': '2024-01-01', 'price': 1299.0},
                {'date': '2024-01-02', 'price': 1299.0},
                # ... 30 days
            ]
        },
        'reviews': [
            {
                'review_id': 'rev_001',
                'reviewer_id': 'user_023',
                'rating': 5,
                'text': 'Great quality fabric, fits perfectly!',
                'is_verified': True,
                'date': '2024-01-15',
                'account_age_days': 450
            },
            # ... more reviews
        ],
        'sizing_data': {
            'size_chart': {
                'S': {'chest': 96, 'waist': 86, 'hips': 98},
                'M': {'chest': 101, 'waist': 91, 'hips': 103},
                'L': {'chest': 106, 'waist': 96, 'hips': 108},
                'XL': {'chest': 111, 'waist': 101, 'hips': 113}
            },
            'fit_feedback': ['true to size', 'true to size', 'runs small']
        }
    }
}
```

### User Profile Schema

Current user profile is stored in session state.

```python
current_user: User = {
    'user_id': 'user_001',
    'name': 'Priya Sharma',
    'location': 'Mumbai',
    'measurements': {
        'height_cm': 162.0,
        'weight_kg': 58.0,
        'chest_cm': 91.0,
        'waist_cm': 76.0,
        'hips_cm': 96.0
    },
    'behavioral_profile': {
        'total_orders': 45,
        'total_returns': 8,
        'return_rate': 0.178,
        'bracketing_tendency': 0.25
    },
    'preferences': {
        'preferred_brands': ['FabIndia', 'Levi\'s', 'Nike'],
        'deal_sensitivity': 0.7
    }
}
```

### Computed Trust Metrics Schema

Trust metrics are computed on-demand and cached in memory.

```python
trust_metrics: Dict[str, dict] = {
    'prod_001': {
        'review_trust_score': 78,
        'components': {
            'sentiment_score': 85.0,
            'verified_ratio': 80.0,
            'account_score': 75.0,
            'burst_penalty': 0.0,
            'pricing_penalty': 0.0
        },
        'trust_badge': 'green',
        'deal_authenticity': {
            'is_authentic_deal': True,
            'claimed_discount': 35.0,
            'actual_discount': 33.5,
            'avg_30day_price': 1320.0
        }
    }
}
```


## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### Property Reflection

After analyzing all acceptance criteria, I identified several opportunities to consolidate redundant properties:

1. **Trust Score Component Properties (2.1, 2.2, 2.4, 2.6)**: These are all validated by the comprehensive trust score formula property (2.5), which tests the weighted calculation including all components and clamping.

2. **Discount Calculation Properties (3.2, 3.3, 3.4, 3.5)**: These mathematical formulas and detection logic are all validated by the higher-level deal authenticity property (3.6).

3. **Sentiment Normalization Properties (9.1, 9.2, 9.3, 9.5, 9.6)**: These normalization steps and aggregations are validated by the sentiment consistency calculation property (9.4) and the overall trust score formula.

4. **Return Risk Components (4.3, 4.7, 4.9)**: The user return rate component, range constraint, and alternative size recommendations are validated by the comprehensive return probability formula (4.6) and sizing feedback property (4.5/5.6).

5. **Fit Calculation Properties (4.1, 5.2, 5.3)**: The fit delta calculation is the same across contexts - consolidated into a single property (5.3).

6. **User Data Completeness (6.3, 6.4)**: Both measurement and behavioral profile completeness can be tested in a single comprehensive property.

7. **UI Rendering Redundancies (7.7, 7.8)**: Conversation history display is validated by context persistence (1.4), and badge display is validated by trust visualization (7.3).

8. **Implementation Constraints (10.3, 10.4, 10.5)**: These describe how components should be implemented but aren't testable as functional properties.

The following properties represent the unique, non-redundant validation requirements:

### Core Properties

**Property 1: Clarifying Questions for Incomplete Criteria**
*For any* user input with incomplete criteria (missing category or budget), the Shopping_Copilot should ask clarifying questions rather than making recommendations.
**Validates: Requirements 1.2**

**Property 2: Recommendation Count Invariant**
*For any* complete criteria set (category + budget), the Shopping_Copilot should return exactly 3 product recommendations.
**Validates: Requirements 1.3**

**Property 3: Conversation Context Persistence**
*For any* multi-turn conversation, information provided in turn N should be accessible and used in turn N+M for all M > 0.
**Validates: Requirements 1.4, 7.7**

**Property 4: Criteria Satisfaction**
*For any* user criteria set (category, budget range, preferences), all recommended products should satisfy every specified constraint.
**Validates: Requirements 1.5**

**Property 5: Trust Score Formula Correctness**
*For any* product with known component values (sentiment_score, verified_ratio, account_score, burst_penalty, pricing_penalty), the computed Review_Trust_Score should equal: (0.35 × sentiment_score) + (0.25 × verified_ratio) + (0.20 × account_score) - (0.15 × burst_penalty) - (0.15 × pricing_penalty), clamped to [0, 100].
**Validates: Requirements 2.1, 2.2, 2.4, 2.5, 2.6, 9.5**

**Property 6: Sentiment Consistency Calculation**
*For any* review with star rating R and sentiment polarity P, the consistency score should equal: 1 - |((P + 1) / 2) - (R / 5)|.
**Validates: Requirements 9.1, 9.2, 9.3, 9.4, 9.6**

**Property 7: Burst Detection Accuracy**
*For any* set of reviews where more than 30% are posted on a single day, the burst_penalty should be 50; if more than 60% are in a 3-day window, burst_penalty should be 30; otherwise 0.
**Validates: Requirements 2.3**

**Property 8: Price History Average Calculation**
*For any* 30-day price history, the calculated average should equal the sum of all prices divided by the count of price entries.
**Validates: Requirements 3.1**

**Property 9: Deal Authenticity Detection**
*For any* product pricing data, when |claimed_discount - actual_discount| > 10 percentage points, is_authentic_deal should be false; otherwise true, where claimed_discount = ((MRP - current_price) / MRP) × 100 and actual_discount = ((avg_30day_price - current_price) / avg_30day_price) × 100.
**Validates: Requirements 3.2, 3.3, 3.4, 3.5, 3.6**

**Property 10: Fit Delta Calculation**
*For any* user measurements and garment size specifications, the fit_delta for each dimension should equal the absolute difference: |user_measurement - garment_measurement|.
**Validates: Requirements 4.1, 5.2, 5.3**

**Property 11: Return Probability Formula Correctness**
*For any* feature vector (bracketing, user_return_rate, category_risk, fit_delta, sizing_feedback), the return probability should equal: (1 / (1 + e^(-logit))) × 100, where logit = -2.0 + 1.5×bracketing + 2.0×user_return_rate + 1.0×category_risk + 1.8×fit_delta + 0.8×sizing_feedback, and the result should be in [0, 100].
**Validates: Requirements 4.3, 4.6, 4.7**

**Property 12: Bracketing Detection Impact**
*For any* user and product, if the cart contains multiple sizes of the same product, the return probability should be higher than if only one size is in the cart (all other factors equal).
**Validates: Requirements 4.2**

**Property 13: Category Risk Baseline**
*For any* Fashion product and Electronics product with identical other risk factors, the Fashion product should have higher return probability due to category baseline (0.4 vs 0.15).
**Validates: Requirements 4.4**

**Property 14: Optimal Size Recommendation**
*For any* user and apparel product, the recommended size should be the one with minimum total_fit_delta (sum of chest_delta + waist_delta + hips_delta) across all available sizes.
**Validates: Requirements 5.5**

**Property 15: Fit Issue Flagging**
*For any* user and apparel product, if any dimension's fit_delta exceeds 5cm, a fit issue warning should be flagged.
**Validates: Requirements 5.4**

**Property 16: Sizing Feedback Adjustment**
*For any* product with dominant "runs small" feedback, if the recommended size is not the largest available, the system should suggest one size up; similarly for "runs large", suggest one size down if not the smallest.
**Validates: Requirements 4.5, 4.9, 5.6**

**Property 17: User Data Completeness**
*For all* generated users, the data structure should contain non-null values for: user_id, name, location, all measurement fields (height, weight, chest, waist, hips), and all behavioral profile fields (total_orders, total_returns, return_rate, bracketing_tendency).
**Validates: Requirements 6.3, 6.4**

**Property 18: Product Schema Compliance**
*For all* generated products, the data structure should conform to Product_Schema with all required fields present and correctly typed.
**Validates: Requirements 6.10**

**Property 19: Review Sentiment Diversity**
*For all* generated products, the review set should include both sentiment-consistent reviews (sentiment matches rating) and sentiment-inconsistent reviews (sentiment mismatches rating) to enable trust score testing.
**Validates: Requirements 6.8**

**Property 20: Price Pattern Diversity**
*For all* generated products, the price histories should include both normal pricing patterns and artificial inflation patterns (MRP increase before sale).
**Validates: Requirements 6.9**

**Property 21: Product Card Rendering Completeness**
*For any* product, the rendered product card should contain all required fields: image, title, brand, price, discount, and trust badge.
**Validates: Requirements 7.2**

**Property 22: Trust Score Visualization**
*For any* product, the rendered detail view should include the Review_Trust_Score as a numerical value and a color-coded visual indicator (green for >70, amber for 40-70, red for <40), with appropriate badge styling.
**Validates: Requirements 2.7, 2.8, 2.9, 7.3, 7.8**

**Property 23: Deceptive Pricing Warning Display**
*For any* product where is_authentic_deal is false, the rendered UI should contain a warning message with actual vs claimed discount comparison.
**Validates: Requirements 3.7, 7.4**

**Property 24: High-Risk Modal Trigger**
*For any* product with return_probability > 60%, attempting to add it to cart should trigger a modal dialog displaying the risk percentage and contributing factors.
**Validates: Requirements 4.8, 7.5**

**Property 25: Apparel Size Recommendation Display**
*For any* Fashion category product, the rendered detail view should include the Smart_Fit_Intelligence size recommendation and fit analysis explanation.
**Validates: Requirements 5.7, 7.6**

**Property 26: Session Data Persistence**
*For any* data generated at application startup, that data should remain accessible and unchanged throughout the entire session duration.
**Validates: Requirements 10.7**

### Edge Case Properties

**Edge Case 1: Empty Conversation Start**
When a user initiates a shopping session with no prior context, the Shopping_Copilot should greet the user and ask about shopping needs.
**Validates: Requirements 1.1**

**Edge Case 2: Vague Query Handling**
When a user provides vague requirements (e.g., "I need something"), the Shopping_Copilot should ask clarifying questions rather than making recommendations.
**Validates: Requirements 1.2**

**Edge Case 3: Trust Score Boundary Classification**
Products with scores of exactly 40 and exactly 70 should be classified correctly at the boundaries (score < 40 → red, 40 ≤ score ≤ 70 → amber, score > 70 → green).
**Validates: Requirements 2.7, 2.8, 2.9**

### Example-Based Properties

**Example 1: Demo Data Counts**
The Demo_Data_Generator should produce exactly 50 users and exactly 200 products.
**Validates: Requirements 6.1, 6.2**

**Example 2: Bad Actor Injection Counts**
The generated data should include at least 20 products with fake reviews, at least 15 with deceptive pricing, and at least 25 apparel items with high return risk.
**Validates: Requirements 6.5, 6.6, 6.7**

**Example 3: Welcome Screen Display**
When the application starts, the UI should display a welcome screen with instructions.
**Validates: Requirements 7.1, 8.1**

**Example 4: Chat Interface Presence**
The Streamlit UI should display a chat interface for conversational interaction.
**Validates: Requirements 7.1**

**Example 5: Complete Flow Demonstration**
The system should support the complete user flow: query → clarification → recommendations → detail view → trust display → cart addition → risk warning.
**Validates: Requirements 8.2**

**Example 6: Demo Data Diversity**
The generated demo data should include at least one high-trust product (score > 70), one low-trust product (score < 40), and one high-return-risk product (probability > 60%).
**Validates: Requirements 8.3**

**Example 7: Standalone Operation**
The system should generate all required data at startup and make no HTTP requests to external APIs during operation.
**Validates: Requirements 10.1, 10.2**

## Error Handling

### Input Validation

1. **User Input Sanitization**: All user messages in the chat interface should be sanitized to prevent injection attacks or malformed input from breaking the conversation flow.

2. **Measurement Validation**: User measurements should be validated to ensure they fall within realistic ranges (e.g., height: 100-250cm, weight: 30-200kg). Invalid measurements should trigger a warning and use default values.

3. **Price Data Validation**: Price history data should be validated to ensure all prices are positive numbers. Missing or invalid price entries should be filtered out before calculating averages.

4. **Review Data Validation**: Reviews with missing required fields (rating, text, date) should be excluded from trust score calculations rather than causing errors.

### Calculation Safeguards

1. **Division by Zero Protection**: All division operations (discount calculations, averages, ratios) should check for zero denominators and handle them gracefully (return 0 or skip calculation).

2. **Sentiment Analysis Fallback**: If sentiment analysis fails for a review (e.g., empty text), assign a neutral sentiment (0.5) rather than crashing.

3. **Missing Size Chart Handling**: If an apparel product lacks a size chart, Smart Fit Intelligence should return a generic recommendation ("Size M") with a warning rather than failing.

4. **Empty Product Set Handling**: If no products match user criteria, the Shopping Copilot should inform the user and ask them to adjust their requirements rather than returning an empty list.

### UI Error States

1. **Loading States**: Display loading indicators during data generation and trust score calculations to prevent user confusion.

2. **Graceful Degradation**: If trust score calculation fails for a product, display the product with a "Trust Score Unavailable" badge rather than hiding it.

3. **Cart Operation Errors**: If adding to cart fails, display an error message and allow the user to retry rather than silently failing.

## Testing Strategy

### Dual Testing Approach

The system will use both unit testing and property-based testing for comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, and error conditions
- **Property tests**: Verify universal properties across all inputs

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across a wide input space.

### Property-Based Testing Configuration

**Library Selection**: Use **Hypothesis** for Python property-based testing.

**Test Configuration**:
- Minimum 100 iterations per property test (due to randomization)
- Each property test must reference its design document property
- Tag format: `# Feature: ai-shopping-copilot, Property {number}: {property_text}`

**Example Property Test Structure**:

```python
from hypothesis import given, strategies as st
import hypothesis

@given(
    sentiment_score=st.floats(min_value=0, max_value=100),
    verified_ratio=st.floats(min_value=0, max_value=100),
    account_score=st.floats(min_value=0, max_value=100),
    burst_penalty=st.floats(min_value=0, max_value=100),
    pricing_penalty=st.floats(min_value=0, max_value=100)
)
@hypothesis.settings(max_examples=100)
def test_trust_score_formula_correctness(
    sentiment_score, verified_ratio, account_score, burst_penalty, pricing_penalty
):
    # Feature: ai-shopping-copilot, Property 4: Trust Score Formula Correctness
    
    expected = (0.35 * sentiment_score) + (0.25 * verified_ratio) + \
               (0.20 * account_score) - (0.15 * burst_penalty) - \
               (0.15 * pricing_penalty)
    expected = max(0, min(100, expected))  # Clamp to [0, 100]
    
    actual = trust_engine.calculate_trust_score({
        'sentiment_score': sentiment_score,
        'verified_ratio': verified_ratio,
        'account_score': account_score,
        'burst_penalty': burst_penalty,
        'pricing_penalty': pricing_penalty
    })
    
    assert abs(actual - expected) < 0.01  # Allow small floating point error
```

### Unit Testing Focus Areas

Unit tests should focus on:

1. **Specific Examples**: 
   - Test welcome screen displays correct greeting message
   - Test that a product with score 75 gets a green badge
   - Test that exactly 50 users are generated

2. **Edge Cases**:
   - Test trust score boundary values (39, 40, 70, 71)
   - Test empty review sets
   - Test products with no price history
   - Test users with extreme measurements

3. **Error Conditions**:
   - Test division by zero in discount calculations
   - Test sentiment analysis with empty text
   - Test missing size chart handling
   - Test invalid user input sanitization

4. **Integration Points**:
   - Test that Shopping Copilot correctly calls Trust Engine
   - Test that UI Controller correctly renders trust badges
   - Test that Return Risk Predictor correctly uses Smart Fit Intelligence

### Test Coverage Goals

- **Core Algorithms**: 100% coverage of Trust Engine, Return Risk Predictor, Smart Fit Intelligence
- **Data Generation**: 100% coverage of Demo Data Generator
- **UI Rendering**: 80% coverage (focus on logic, not visual styling)
- **Conversation Logic**: 90% coverage of Shopping Copilot

### Testing Workflow

1. **Development Phase**: Write property tests alongside implementation for each component
2. **Integration Phase**: Write unit tests for integration points and edge cases
3. **Demo Preparation**: Run full test suite to ensure all properties hold
4. **Continuous Validation**: Re-run tests after any code changes

### Mock Data for Testing

For unit tests, create minimal mock data:
- 5 mock users with known measurements
- 10 mock products with controlled review patterns
- Known price histories with specific patterns

For property tests, use Hypothesis strategies to generate:
- Random user measurements within realistic ranges
- Random product data with varying review patterns
- Random price histories with different patterns
- Random conversation states with varying criteria

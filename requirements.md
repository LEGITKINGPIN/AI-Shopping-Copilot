# Requirements Document: AI Shopping Copilot for Retail

## Introduction

The AI Shopping Copilot is a Trust-Driven Decision Intelligence Layer designed for Indian e-commerce platforms. The system addresses three critical challenges: fake reviews (60% of reviews are bot-generated), deceptive pricing practices (inflated MRP before sales), and high return rates (40% in fashion due to poor size standardization). The copilot combines conversational AI for product discovery, algorithmic trust scoring for review authenticity, pricing manipulation detection, and predictive analytics for return risk assessment.

## Glossary

- **Shopping_Copilot**: The conversational AI agent that assists users in product discovery and purchase decisions
- **Trust_Engine**: The algorithmic system that calculates review authenticity and pricing integrity scores
- **Review_Trust_Score**: A numerical score (0-100) indicating the reliability of product reviews
- **Deal_Authenticity_Analyzer**: Component that detects fake discounts by analyzing price history
- **Return_Risk_Predictor**: Machine learning system that predicts likelihood of product returns
- **Smart_Fit_Intelligence**: Sizing recommendation system for apparel based on body measurements
- **Bracketing**: User behavior of ordering multiple sizes with intent to return unwanted items
- **Burst_Detection**: Algorithm that identifies suspicious temporal patterns in review posting
- **Fit_Delta**: Numerical difference between user measurements and garment size specifications
- **Demo_Data_Generator**: System component that creates synthetic test data for demonstration
- **User_Profile**: Data structure containing user preferences, measurements, and behavioral history
- **Product_Schema**: Data structure defining product attributes, pricing, reviews, and trust metrics
- **Streamlit_UI**: The Python-based frontend framework for the application interface

## Requirements

### Requirement 1: Conversational Shopping Assistant

**User Story:** As a shopper, I want to have a natural conversation about my needs, so that I can discover products that truly match my requirements without browsing hundreds of listings.

#### Acceptance Criteria

1. WHEN a user initiates a shopping session, THE Shopping_Copilot SHALL greet the user and ask about their shopping needs
2. WHEN a user provides vague requirements, THE Shopping_Copilot SHALL ask clarifying questions about budget, use-case, and preferences
3. WHEN the Shopping_Copilot has gathered sufficient information, THE Shopping_Copilot SHALL recommend exactly three products that match the criteria
4. WHEN a conversation spans multiple turns, THE Shopping_Copilot SHALL maintain context of all previous user inputs
5. WHEN generating recommendations, THE Shopping_Copilot SHALL synthesize all user-provided criteria including budget constraints, category preferences, and use-case requirements

### Requirement 2: Review Trust Score Calculation

**User Story:** As a shopper, I want to know if product reviews are genuine, so that I can make informed decisions without being misled by fake reviews.

#### Acceptance Criteria

1. WHEN calculating Review_Trust_Score, THE Trust_Engine SHALL analyze sentiment consistency between star ratings and review text
2. WHEN calculating Review_Trust_Score, THE Trust_Engine SHALL factor in the ratio of verified purchases to total reviews
3. WHEN calculating Review_Trust_Score, THE Trust_Engine SHALL detect temporal burst patterns indicating bot-generated reviews
4. WHEN calculating Review_Trust_Score, THE Trust_Engine SHALL evaluate reviewer account credibility scores
5. THE Trust_Engine SHALL compute Review_Trust_Score using weighted formula: RTS = (0.35 × Sentiment_Score) + (0.25 × Verified_Ratio) + (0.20 × Account_Score) - (0.15 × Burst_Penalty) - (0.15 × Pricing_Penalty)
6. THE Trust_Engine SHALL output Review_Trust_Score as an integer value between 0 and 100
7. WHEN Review_Trust_Score is below 40, THE Trust_Engine SHALL classify the product as "Low Trust" with red badge
8. WHEN Review_Trust_Score is between 40 and 70, THE Trust_Engine SHALL classify the product as "Medium Trust" with amber badge
9. WHEN Review_Trust_Score is above 70, THE Trust_Engine SHALL classify the product as "High Trust" with green badge

### Requirement 3: Deal Authenticity Analysis

**User Story:** As a shopper, I want to know if advertised discounts are genuine, so that I am not deceived by artificially inflated prices.

#### Acceptance Criteria

1. WHEN analyzing deal authenticity, THE Deal_Authenticity_Analyzer SHALL calculate 30-day moving average of product prices
2. WHEN a product's MRP increased within 30 days before a sale, THE Deal_Authenticity_Analyzer SHALL flag the deal as potentially deceptive
3. WHEN calculating actual discount, THE Deal_Authenticity_Analyzer SHALL compare current price against 30-day average price rather than listed MRP
4. THE Deal_Authenticity_Analyzer SHALL compute claimed_discount as ((MRP - current_price) / MRP) × 100
5. THE Deal_Authenticity_Analyzer SHALL compute actual_discount as ((avg_30day_price - current_price) / avg_30day_price) × 100
6. WHEN actual_discount differs from claimed_discount by more than 10 percentage points, THE Deal_Authenticity_Analyzer SHALL mark is_authentic_deal as false
7. WHEN is_authentic_deal is false, THE Streamlit_UI SHALL display a warning message indicating pricing manipulation

### Requirement 4: Return Risk Prediction

**User Story:** As a shopper, I want to know if a product is likely to fit me correctly, so that I can avoid the hassle of returns and exchanges.

#### Acceptance Criteria

1. WHEN predicting return risk for apparel, THE Return_Risk_Predictor SHALL calculate Fit_Delta between user measurements and garment size chart
2. WHEN a user has items of multiple sizes in cart, THE Return_Risk_Predictor SHALL detect bracketing behavior and increase risk score
3. WHEN calculating return risk, THE Return_Risk_Predictor SHALL factor in user's historical return rate
4. WHEN calculating return risk, THE Return_Risk_Predictor SHALL incorporate category baseline return rates
5. WHEN product reviews contain sizing feedback signals, THE Return_Risk_Predictor SHALL adjust risk based on "runs small", "true to size", or "runs large" patterns
6. THE Return_Risk_Predictor SHALL compute return probability using logistic regression: P = (1 / (1 + e^(-logit))) × 100
7. THE Return_Risk_Predictor SHALL output return probability as integer percentage between 0 and 100
8. WHEN return probability exceeds 60%, THE Streamlit_UI SHALL display a warning modal before cart addition
9. WHEN return probability exceeds 60% for apparel, THE Smart_Fit_Intelligence SHALL recommend alternative sizes with lower risk scores

### Requirement 5: Smart Fit Intelligence for Apparel

**User Story:** As a shopper buying clothing, I want personalized size recommendations based on my measurements, so that I can order the correct size on the first attempt.

#### Acceptance Criteria

1. WHEN a user views an apparel product, THE Smart_Fit_Intelligence SHALL retrieve user body measurements from User_Profile
2. WHEN calculating fit recommendation, THE Smart_Fit_Intelligence SHALL compare user measurements against product size chart for chest, waist, and hip dimensions
3. THE Smart_Fit_Intelligence SHALL calculate Fit_Delta as absolute difference between user measurement and garment specification for each dimension
4. WHEN Fit_Delta for any dimension exceeds 5cm, THE Smart_Fit_Intelligence SHALL flag potential fit issue
5. THE Smart_Fit_Intelligence SHALL recommend the size with minimum total Fit_Delta across all dimensions
6. WHEN multiple sizes have similar Fit_Delta, THE Smart_Fit_Intelligence SHALL recommend based on review sizing feedback patterns
7. THE Streamlit_UI SHALL display recommended size with visual indicator and explanation of fit analysis

### Requirement 6: Demo Data Generation

**User Story:** As a developer, I want synthetic demo data that realistically simulates e-commerce scenarios including bad actors, so that I can demonstrate the system's capabilities without external dependencies.

#### Acceptance Criteria

1. THE Demo_Data_Generator SHALL create exactly 50 synthetic users with Indian names and locations using Faker library
2. THE Demo_Data_Generator SHALL generate exactly 200 products split between Electronics and Fashion categories
3. WHEN generating user data, THE Demo_Data_Generator SHALL include physical measurements (height, weight, chest, waist, hips) for all users
4. WHEN generating user data, THE Demo_Data_Generator SHALL assign behavioral risk profiles including total orders, returns, and bracketing tendency
5. THE Demo_Data_Generator SHALL inject at least 20 products with fake reviews exhibiting generic 5-star text patterns
6. THE Demo_Data_Generator SHALL inject at least 15 products with deceptive pricing showing MRP inflation before sales
7. THE Demo_Data_Generator SHALL inject at least 25 apparel products with high return risk due to poor fit characteristics
8. WHEN generating reviews, THE Demo_Data_Generator SHALL create sentiment patterns that match or mismatch star ratings for trust score testing
9. WHEN generating price history, THE Demo_Data_Generator SHALL create 30-day price sequences including artificial inflation patterns
10. THE Demo_Data_Generator SHALL store all generated data in Product_Schema and User_Profile formats compatible with Trust_Engine and Return_Risk_Predictor

### Requirement 7: Visual Dashboard and User Interface

**User Story:** As a shopper, I want a clean and intuitive interface that clearly shows trust indicators and risk warnings, so that I can quickly assess product reliability.

#### Acceptance Criteria

1. THE Streamlit_UI SHALL display a chat interface for conversational interaction with Shopping_Copilot
2. WHEN displaying product recommendations, THE Streamlit_UI SHALL render product cards showing image, title, brand, price, and discount
3. WHEN displaying a product, THE Streamlit_UI SHALL show Review_Trust_Score as a visual gauge with color coding (green/amber/red)
4. WHEN a product has deceptive pricing, THE Streamlit_UI SHALL display a warning banner with actual vs claimed discount comparison
5. WHEN a user adds high-risk product to cart, THE Streamlit_UI SHALL show a modal dialog with return probability and fit analysis
6. WHEN displaying apparel products, THE Streamlit_UI SHALL show Smart_Fit_Intelligence recommendation with size chart comparison
7. THE Streamlit_UI SHALL maintain conversation history visible in the chat interface throughout the session
8. THE Streamlit_UI SHALL use visual badges (green checkmark, amber caution, red warning) to indicate trust levels at a glance

### Requirement 8: Complete User Flow Demonstration

**User Story:** As a hackathon judge, I want to see a complete end-to-end shopping journey, so that I can evaluate the system's practical value and user experience.

#### Acceptance Criteria

1. WHEN the application starts, THE Streamlit_UI SHALL display a welcome screen with clear instructions for beginning a shopping session
2. THE system SHALL support the following complete flow: user query → AI clarification → product recommendations → product detail view → trust score display → cart addition → return risk warning
3. WHEN demonstrating the flow, THE system SHALL showcase at least one high-trust product, one low-trust product, and one high-return-risk product
4. THE system SHALL execute the complete flow without runtime errors or missing data
5. THE system SHALL complete each conversation turn within 3 seconds for responsive user experience
6. WHEN the demo completes, THE system SHALL have demonstrated all three core pillars: Decision Intelligence, Trust Engine, and Risk Reduction

### Requirement 9: Sentiment Consistency Analysis

**User Story:** As a system administrator, I want the Trust Engine to detect mismatches between review ratings and text sentiment, so that fake reviews can be identified algorithmically.

#### Acceptance Criteria

1. WHEN analyzing a review, THE Trust_Engine SHALL extract sentiment polarity from review text using sentiment analysis
2. THE Trust_Engine SHALL normalize sentiment polarity to a 0-1 scale where 0 is most negative and 1 is most positive
3. THE Trust_Engine SHALL normalize star rating to a 0-1 scale by dividing by 5
4. THE Trust_Engine SHALL calculate sentiment consistency as 1 minus absolute difference between normalized sentiment and normalized rating
5. WHEN sentiment consistency is below 0.5, THE Trust_Engine SHALL apply a penalty to Review_Trust_Score
6. THE Trust_Engine SHALL aggregate sentiment consistency across all reviews to compute product-level Sentiment_Score component

### Requirement 10: Standalone Operation

**User Story:** As a hackathon participant, I want the system to run completely offline without external API dependencies, so that I can demonstrate it reliably in any environment.

#### Acceptance Criteria

1. THE system SHALL generate all required data using Demo_Data_Generator at startup
2. THE system SHALL NOT make HTTP requests to external APIs during normal operation
3. THE Shopping_Copilot SHALL use rule-based logic or local models for conversation management
4. THE Trust_Engine SHALL perform all calculations using local Python algorithms
5. THE Return_Risk_Predictor SHALL use locally implemented logistic regression without external ML services
6. WHEN the application starts, THE system SHALL complete initialization and data generation within 10 seconds
7. THE system SHALL persist generated demo data in memory for the duration of the session

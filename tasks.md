# Implementation Plan: AI Shopping Copilot for Retail

## Overview

This implementation plan breaks down the AI Shopping Copilot into discrete coding tasks that build incrementally. The system will be implemented in Python using Streamlit for the UI, with three core subsystems: Shopping Copilot (conversational AI), Trust Engine (review and pricing analysis), and Return Risk Predictor (fit analysis and risk scoring). Each task builds on previous work, with testing integrated throughout to validate correctness early.

## Tasks

- [ ] 1. Set up project structure and dependencies
  - Create project directory structure: `src/`, `tests/`, `data/`
  - Create `requirements.txt` with dependencies: streamlit, faker, textblob, numpy, pandas, hypothesis, pytest
  - Create main entry point `app.py` for Streamlit application
  - Set up basic Streamlit app skeleton with title and welcome message
  - _Requirements: 10.1, 10.2_

- [ ] 2. Implement Demo Data Generator
  - [ ] 2.1 Create data models and schemas
    - Define Python dataclasses or dictionaries for User, Product, Review, SizingData schemas
    - Implement validation functions for schema compliance
    - _Requirements: 6.10_
  
  - [ ]* 2.2 Write property test for schema compliance
    - **Property 17: Product Schema Compliance**
    - **Validates: Requirements 6.10**
  
  - [ ] 2.3 Implement user generation
    - Use Faker to generate 50 users with Indian names and locations
    - Generate realistic body measurements using normal distributions
    - Assign behavioral profiles with varying return rates and bracketing tendencies
    - _Requirements: 6.1, 6.3, 6.4_
  
  - [ ]* 2.4 Write property test for user data completeness
    - **Property 16: User Data Completeness**
    - **Validates: Requirements 6.3, 6.4**
  
  - [ ] 2.5 Implement product generation
    - Generate 200 products (120 Electronics, 80 Fashion)
    - Create realistic titles, brands, and pricing data
    - Generate 30-day price histories with normal and inflated patterns
    - _Requirements: 6.2, 6.9_
  
  - [ ]* 2.6 Write property test for price pattern diversity
    - **Property 19: Price Pattern Diversity**
    - **Validates: Requirements 6.9**
  
  - [ ] 2.7 Implement review generation
    - Generate 5-20 reviews per product with varying patterns
    - Create sentiment-consistent and sentiment-inconsistent reviews
    - Inject fake review patterns (generic 5-star text, temporal bursts)
    - _Requirements: 6.5, 6.8_
  
  - [ ]* 2.8 Write property test for review sentiment diversity
    - **Property 18: Review Sentiment Diversity**
    - **Validates: Requirements 6.8**
  
  - [ ] 2.9 Implement bad actor injection
    - Inject 20+ products with fake reviews
    - Inject 15+ products with deceptive pricing
    - Inject 25+ apparel products with poor fit characteristics
    - _Requirements: 6.5, 6.6, 6.7_
  
  - [ ]* 2.10 Write unit tests for bad actor counts
    - Test that generated data includes minimum required bad actor products
    - _Requirements: 6.5, 6.6, 6.7_

- [ ] 3. Checkpoint - Verify data generation
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 4. Implement Trust Engine
  - [ ] 4.1 Implement sentiment consistency analysis
    - Integrate TextBlob for sentiment polarity extraction
    - Implement sentiment and rating normalization to 0-1 scale
    - Calculate consistency score for each review
    - Aggregate to product-level Sentiment_Score
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.6_
  
  - [ ]* 4.2 Write property test for sentiment consistency calculation
    - **Property 5: Sentiment Consistency Calculation**
    - **Validates: Requirements 9.1, 9.2, 9.3, 9.4**
  
  - [ ] 4.3 Implement burst detection
    - Group reviews by date
    - Detect single-day bursts (>30% of reviews)
    - Detect 3-day window bursts (>60% of reviews)
    - Calculate burst_penalty (0, 30, or 50)
    - _Requirements: 2.3_
  
  - [ ]* 4.4 Write property test for burst detection accuracy
    - **Property 6: Burst Detection Accuracy**
    - **Validates: Requirements 2.3**
  
  - [ ] 4.5 Implement verified ratio and account score calculations
    - Calculate verified purchase ratio
    - Calculate average account credibility from account ages
    - _Requirements: 2.2, 2.4_
  
  - [ ] 4.6 Implement pricing penalty calculation
    - Calculate 30-day average price from price history
    - Detect MRP inflation (MRP > avg_price × 1.15)
    - Set pricing_penalty to 40 if inflated, else 0
    - _Requirements: 3.1_
  
  - [ ]* 4.7 Write property test for price history average
    - **Property 8: Price History Average Calculation**
    - **Validates: Requirements 3.1**
  
  - [ ] 4.8 Implement Review Trust Score formula
    - Combine all components using weighted formula
    - Clamp result to [0, 100] range
    - Return score and component breakdown
    - _Requirements: 2.5, 2.6_
  
  - [ ]* 4.9 Write property test for trust score formula
    - **Property 4: Trust Score Formula Correctness**
    - **Validates: Requirements 2.5, 2.6**
  
  - [ ] 4.10 Implement trust badge classification
    - Map score ranges to badge colors (red <40, amber 40-70, green >70)
    - _Requirements: 2.7, 2.8, 2.9_
  
  - [ ]* 4.11 Write unit tests for trust score boundary classification
    - Test edge cases at scores 39, 40, 70, 71
    - _Requirements: 2.7, 2.8, 2.9_
  
  - [ ] 4.12 Implement Deal Authenticity Analyzer
    - Calculate claimed_discount from MRP and current_price
    - Calculate actual_discount from 30-day average and current_price
    - Set is_authentic_deal based on 10 percentage point threshold
    - _Requirements: 3.2, 3.3, 3.4, 3.5, 3.6_
  
  - [ ]* 4.13 Write property test for deal authenticity detection
    - **Property 7: Deal Authenticity Detection**
    - **Validates: Requirements 3.2, 3.3, 3.6**

- [ ] 5. Checkpoint - Verify trust engine
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 6. Implement Smart Fit Intelligence
  - [ ] 6.1 Implement fit delta calculation
    - Calculate absolute difference for chest, waist, hips dimensions
    - Sum deltas to get total_fit_delta for each size
    - _Requirements: 4.1, 5.2, 5.3_
  
  - [ ]* 6.2 Write property test for fit delta calculation
    - **Property 12: Fit Delta Calculation**
    - **Validates: Requirements 4.1, 5.3**
  
  - [ ] 6.3 Implement size recommendation logic
    - Find size with minimum total_fit_delta
    - Flag fit issues when any dimension delta > 5cm
    - _Requirements: 5.4, 5.5_
  
  - [ ]* 6.4 Write property test for optimal size recommendation
    - **Property 13: Optimal Size Recommendation**
    - **Validates: Requirements 5.5**
  
  - [ ]* 6.5 Write property test for fit issue flagging
    - **Property 14: Fit Issue Flagging**
    - **Validates: Requirements 5.4**
  
  - [ ] 6.6 Implement fit feedback analysis
    - Parse review fit_feedback for "runs small", "true to size", "runs large"
    - Determine dominant pattern
    - Adjust size recommendation based on feedback
    - _Requirements: 5.6_
  
  - [ ]* 6.7 Write property test for sizing feedback adjustment
    - **Property 15: Sizing Feedback Adjustment**
    - **Validates: Requirements 5.6, 4.5**
  
  - [ ] 6.8 Implement fit explanation generator
    - Generate human-readable explanation of fit analysis
    - Include recommended size, fit deltas, and reasoning
    - _Requirements: 5.7_

- [ ] 7. Implement Return Risk Predictor
  - [ ] 7.1 Implement feature extraction
    - Extract bracketing feature from cart context
    - Extract user_return_rate from user profile
    - Extract category_risk (0.4 for Fashion, 0.15 for Electronics)
    - Calculate normalized fit_delta from Smart Fit Intelligence
    - Extract sizing_feedback adjustment from reviews
    - _Requirements: 4.2, 4.3, 4.4, 4.5_
  
  - [ ] 7.2 Implement logistic regression calculation
    - Calculate logit using coefficients: β0=-2.0, β1=1.5, β2=2.0, β3=1.0, β4=1.8, β5=0.8
    - Apply sigmoid function: P = (1 / (1 + e^(-logit))) × 100
    - Clamp result to [0, 100]
    - _Requirements: 4.6, 4.7_
  
  - [ ]* 7.3 Write property test for return probability formula
    - **Property 9: Return Probability Formula Correctness**
    - **Validates: Requirements 4.6, 4.7**
  
  - [ ]* 7.4 Write property test for bracketing detection impact
    - **Property 10: Bracketing Detection Impact**
    - **Validates: Requirements 4.2**
  
  - [ ]* 7.5 Write property test for category risk baseline
    - **Property 11: Category Risk Baseline**
    - **Validates: Requirements 4.4**
  
  - [ ] 7.6 Implement alternative size recommendations
    - When return probability > 60%, find alternative sizes with lower risk
    - Return list of alternatives with their risk scores
    - _Requirements: 4.9_

- [ ] 8. Implement Shopping Copilot
  - [ ] 8.1 Create conversation state management
    - Define ConversationState data structure
    - Implement state initialization and updates
    - Maintain conversation history
    - _Requirements: 1.4_
  
  - [ ]* 8.2 Write property test for conversation context persistence
    - **Property 2: Conversation Context Persistence**
    - **Validates: Requirements 1.4**
  
  - [ ] 8.3 Implement criteria extraction
    - Parse user messages for category, budget, use-case, preferences
    - Update collected_criteria in conversation state
    - _Requirements: 1.5_
  
  - [ ] 8.4 Implement clarification logic
    - Check if criteria is complete (category + budget minimum)
    - Generate targeted clarifying questions for missing criteria
    - _Requirements: 1.2_
  
  - [ ]* 8.5 Write unit test for vague query handling
    - Test that vague inputs trigger clarifying questions
    - _Requirements: 1.2_
  
  - [ ] 8.6 Implement product recommendation logic
    - Filter products by category and budget range
    - Sort by Review Trust Score (descending)
    - Return top 3 products
    - _Requirements: 1.3, 1.5_
  
  - [ ]* 8.7 Write property test for recommendation count invariant
    - **Property 1: Recommendation Count Invariant**
    - **Validates: Requirements 1.3**
  
  - [ ]* 8.8 Write property test for criteria satisfaction
    - **Property 3: Criteria Satisfaction**
    - **Validates: Requirements 1.5**
  
  - [ ] 8.9 Implement conversation flow orchestration
    - Process user message and update state
    - Decide whether to clarify or recommend
    - Generate appropriate response
    - _Requirements: 1.1, 1.2, 1.3_
  
  - [ ]* 8.10 Write unit test for conversation start
    - Test that initial message is a greeting
    - _Requirements: 1.1_

- [ ] 9. Checkpoint - Verify core logic
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. Implement Streamlit UI Controller
  - [ ] 10.1 Set up session state management
    - Initialize session state with conversation_state, current_user, products, cart
    - Implement session state persistence across reruns
    - _Requirements: 10.7_
  
  - [ ]* 10.2 Write property test for session data persistence
    - **Property 25: Session Data Persistence**
    - **Validates: Requirements 10.7**
  
  - [ ] 10.3 Implement chat interface
    - Display conversation history with message bubbles
    - Render text input for user messages
    - Handle user input submission
    - Auto-scroll to latest message
    - _Requirements: 7.1, 7.7_
  
  - [ ] 10.4 Implement product card rendering
    - Display product image (placeholder), title, brand, price
    - Show trust badge with color coding
    - Add "View Details" button
    - _Requirements: 7.2, 7.8_
  
  - [ ]* 10.5 Write property test for product card completeness
    - **Property 20: Product Card Rendering Completeness**
    - **Validates: Requirements 7.2**
  
  - [ ] 10.6 Implement trust score visualization
    - Create gauge component for trust score (0-100)
    - Apply color gradient (red → amber → green)
    - Display component breakdown
    - _Requirements: 7.3_
  
  - [ ]* 10.7 Write property test for trust score visualization
    - **Property 21: Trust Score Visualization**
    - **Validates: Requirements 7.3, 2.7, 2.8, 2.9**
  
  - [ ] 10.8 Implement pricing warning display
    - Check is_authentic_deal flag
    - Display warning banner with actual vs claimed discount
    - _Requirements: 3.7, 7.4_
  
  - [ ]* 10.9 Write property test for deceptive pricing warning
    - **Property 22: Deceptive Pricing Warning Display**
    - **Validates: Requirements 3.7, 7.4**
  
  - [ ] 10.10 Implement product detail view
    - Display full product information
    - Show trust score gauge and breakdown
    - Show pricing authenticity section
    - For apparel: show size recommendation and fit analysis
    - Display reviews
    - Add "Add to Cart" button
    - _Requirements: 7.3, 7.4, 7.6_
  
  - [ ]* 10.11 Write property test for apparel size recommendation display
    - **Property 24: Apparel Size Recommendation Display**
    - **Validates: Requirements 5.7, 7.6**
  
  - [ ] 10.12 Implement return risk modal
    - Trigger modal when adding product with risk > 60%
    - Display return probability percentage
    - Show contributing factors (fit issues, user history, bracketing)
    - Show alternative size recommendations if applicable
    - Add "Proceed Anyway" and "Cancel" buttons
    - _Requirements: 4.8, 7.5_
  
  - [ ]* 10.13 Write property test for high-risk modal trigger
    - **Property 23: High-Risk Modal Trigger**
    - **Validates: Requirements 4.8, 7.5**
  
  - [ ] 10.14 Implement welcome screen
    - Display welcome message and instructions on app start
    - Show example queries to guide users
    - _Requirements: 7.1, 8.1_

- [ ] 11. Wire all components together
  - [ ] 11.1 Integrate Demo Data Generator with app initialization
    - Call data generator on first app load
    - Store generated data in session state
    - Select random user as current_user
    - _Requirements: 10.1_
  
  - [ ] 11.2 Connect Shopping Copilot to chat interface
    - Process user messages through Shopping Copilot
    - Display copilot responses in chat
    - Update conversation state
    - _Requirements: 1.1, 1.2, 1.3_
  
  - [ ] 11.3 Connect Trust Engine to product display
    - Calculate trust metrics for all products
    - Cache results in session state
    - Display trust scores and badges in UI
    - _Requirements: 2.5, 2.6, 7.3_
  
  - [ ] 11.4 Connect Return Risk Predictor to cart operations
    - Calculate return risk when user views apparel products
    - Trigger modal when adding high-risk products
    - Display alternative recommendations
    - _Requirements: 4.6, 4.8, 4.9_
  
  - [ ] 11.5 Connect Smart Fit Intelligence to apparel display
    - Calculate size recommendation for apparel products
    - Display fit analysis in product detail view
    - _Requirements: 5.5, 5.7_
  
  - [ ]* 11.6 Write integration tests for complete user flow
    - Test query → clarification → recommendations → detail → cart flow
    - _Requirements: 8.2_

- [ ] 12. Final checkpoint and polish
  - [ ] 12.1 Add error handling
    - Implement input validation and sanitization
    - Add division by zero protection
    - Handle missing data gracefully
    - Add loading states
    - _Requirements: Error Handling section_
  
  - [ ] 12.2 Verify demo data diversity
    - Confirm at least one high-trust, one low-trust, one high-risk product
    - _Requirements: 8.3_
  
  - [ ] 12.3 Test standalone operation
    - Verify no external API calls
    - Confirm all data generated locally
    - _Requirements: 10.1, 10.2_
  
  - [ ] 12.4 Final testing and validation
    - Run complete test suite
    - Test complete user flow manually
    - Verify all three core pillars demonstrated
    - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional property-based and unit tests that can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation at key milestones
- Property tests validate universal correctness properties using Hypothesis library (minimum 100 iterations each)
- Unit tests validate specific examples, edge cases, and error conditions
- The implementation builds incrementally: data generation → core algorithms → UI → integration
- All components are designed to work offline without external API dependencies

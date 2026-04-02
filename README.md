# KwikChat

KwikChat is a social-commerce concierge prototype. The product concept is an AI agent that lives inside a chat thread and takes a returning shopper from identity resolution to purchase without forcing them out of the conversation.

## Setup

This prototype is intended to be lightweight and demo-friendly. 
Visual reference:
- Stitch prototype (For New User): [KwikChat UI reference](https://stitch.withgoogle.com/preview/5496116782317696755?node-id=c6cf254c1c4c40c7be68f951bd2b7932)
    For Existing buyers: [KwikChat UI reference](https://stitch.withgoogle.com/preview/5496116782317696755?node-id=cd28523254fb4815805a9b771f0607ef)

### CTA in prototype
-> Potential Matches 
-> View Deal (1st product) 
-> Lock in best deal (Size and merchant are preselected) 
-> Apply reward and proceed to checkout 
-> Pay


The intended journey covers five moments:
1. Login and identity recognition
2. Shop the look from image and/or text
3. Best-price comparison across merchants
4. Loyalty redemption inside chat
5. Prefilled checkout generation


## End-to-End Flow

DM (Initiation) → Identity → Discovery → Pricing → Rewards → Checkout

### 1. Login and Identity

Input:
    social handle for repeat users
    lightweight OTP based verification for new user

Output:
    recognized shopper
    loyalty tier
    reward balance
    account creation for new users

### 2. Product Discovery

Input:
    uploaded look image and/or freeform text

Output:
    extracted attributes such as silhouette, palette, occasion, fit
    3 to 5 ranked catalog products
    one-sentence reasoning per recommendation

### 3. Best Price

Input:
    selected product

Output:
    merchant comparison
    applied coupon
    cashback
    effective price
    recommended merchant

### 4. Loyalty Apply

Input:
    shopper balance
    selected product and best offer

Output:
    amount redeemed
    remaining balance(save in backend)
    updated payable amount

### 5. Checkout
Input:
    identity context
    product
    best merchant offer
    rewards applied

Output:
    prefilled address for repeat buyers
    Address form for new user
    locked summary screen or checkout URL

## Edge Case
I have purposely considered different edge cases, other than the ones shared in the assignment.
1. Poor Image Quality - The agent can't identify the product due to low quality picture, as per quality of extraction meta - data.
    Sol. - Graceful Failure. "I can't quite see that—could you send a clearer photo or tell me the brand name?"

2. The Zero Result Sitation - Agent unable to retrieve any product based on user's input.
    Sol - Based on semantic similarity. Show items user might like instead to keep the impulse alive. Never show an empty state. 

3. Session Loss: The user closes Instagram and comes back 2 hours later.
    Sol - Persistent State via Redis. The agent should greet them for their last order, or open cart, keeping the context maintained and experience see



## Architecture Overview

The prototype should be treated as a thin orchestration layer over mocked commerce services.

### 1. Chat UI Layer

Responsibilities:
    Render the DM-like interface based on the Stitch design reference
    Capture handle, OTP, image upload, style text, product selection, and loyalty actions
    Stream assistant turns in a conversational format


### 2. Orchestration Layer

Responsibilities:
    Maintain shopper session state
    Decide which tool to call next
    Merge identity context, catalog retrieval, pricing logic, and checkout prep into one thread


### 3.  Service and Agentic layers
    1. Identity Agent: match user handle or account creation via OTP to a shopper profile
    
    2. Catalog agent: rank products against input text or photo (Use LLM model to check for best result as per the input across merchants)
    
    3. Loyalty agent: compute redeemable balance and adjusted payable amount
    
    4. Checkout agent: generate a locked, prefilled checkout summary with URL

### 4. LLM + Vision Layer

This is where the prototype demonstrates AI reasoning.

Capabilities:
    Interpret shopper intent from chat text
    Parse a reference image into visual attributes
    Retrieve or rank products from the mocked catalog
    Explain price recommendations clearly
    Convert a multi-step flow into concise, trustworthy and personalised assistant chat

Suggested model split:
    Vision-capable model for image understanding plus structured extraction
    Fast text model for response generation and stateful orchestration



## LLM Prompts

### Prompt 1: Base UI
    Let’s start simple. I want you to design a mobile chat interface that looks and feels like Instagram DMs. Keep it clean and modern.

    Include:
    User messages on the right, AI messages on the left, and A typing indicator for the AI. Also add bottom input bar with text input and image upload icon

    add the first state where User sends “Where can I buy this? with the a pic of hoodie
    - AI is typing a response

### Prompt 2: Identity Screen
    Now build on this chat UI.I want to show how we recognise the user.

    Create two scenarios:

    1. Returning user:
    AI greets them personally and Show a small loyalty badge

    2. New user:
    Ask for phone number inside the chat, followed by OTP input. Make sure everything happens inside the chat flow, no full page transitions. Once OTP is successful, move naturally to the next step.

### Prompt 3: Search result
    Now show the results. I want 3 product options displayed inside the chat. Design them as horizontal scrollable cards.

    Each product card should have image, name, price and a CTA like “View Deal”
    Make sure tapping a product view deal  leads to the next step.

    Below the cards, add quick reply buttons - “Show more” and "Find best price"


### Prompt 4: Multiple Merchant pricing screen
    When a user selects a product, show them the available size option along with the best deal across merchants. Design a comparison list Highlighting the BEST DEAL clearly 


### Prompt 5: Loyalty Application
    Now there will be two case here for the loyalty rewards or personalised rewards

    1. Returning buyer
    For the case, when the buyer is a repeat buyer, personalised reward from the selected merchant in the screen and also highlight the loyalty reward points that buyer had earned from their previous purchases. Show the calculations clearly in the costing of the product and add the apply reward and proceed to check out CTA.

    2. New buyer
    For the case, when the buyer is a first time buyer only highlight the personalised reward from the selected merchant, as they have no loyalty reward points. Do the calculations accordingly and show the CTA similar to the first case.

    Make sure that all the screens that we have created so far stays within a chat thread, the UI should feel like it's following a chat thread where everything and every action is being taken place within the inbox.

### Prompt 6: Final Checkout Summary
Now create the checkout screen.There will be two scenarios in the checkout screen.


    1. Repeat user
    In case of the repeat users, create one section of shipping address where it will be pre-filled with the address saved with the selected merchant, along with an edit added button, where they can choose to edit the address. Below this section, there will be a payment section where the previously selected payment method will be pre-selected with the details saved. Also there will be a CTA that will allow user to change the payment method. Below this highlight the order items and the order summary at the bottom, show the payment CTA along with the amount on it.

    2. New user
    In case of new user, create two screens, first will be to take the address where a form will be shown that will ask for all the details required for the delivery of the product with the CTA to save the address. Next screen will be for the payment method with user can can choose the payment method as per their choice. And add the CTA for payment confirmation.

Lastly, create a order confirmation screen, which shows the order is created with the CTA to track the order or to continue shopping.

Make sure all the screens that we have created so far stays in a chat thread and should feel like a conversation to the user/buyer.



### Deterministic VS LLM Logic

Not everything should be delegated to a model.

Deterministic logic that should remain in code:
    OTP validation
    shopper lookup
    effective price computation
    loyalty balance caps
    final payable calculation
    checkout URL or payload assembly

Use the model for:
    image interpretation
    style summarization
    product ranking rationale
    assistant phrasing
    recovery copy for edge cases


## Future Improvements

Potential v2 additions:
    1. Shop the look / Multiple image parsing - shopping all the product in the pic, instead of just a single product, given that they fall in same category
    2. merchant-specific shipping and delivery estimation
    3. richer post-purchase loop including returns and reorder journeys
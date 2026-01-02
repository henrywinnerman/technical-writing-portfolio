# Getting Started with Klaviyo Email Marketing on Shopify

A step-by-step tutorial for setting up automated email marketing that converts browsers into buyers.

## What You'll Achieve

By the end of this tutorial, you'll have:
- Klaviyo integrated with your Shopify store
- An automated welcome email series
- An abandoned cart recovery flow
- Your first email campaign ready to send

**Why Klaviyo?** Over 117,000 brands use Klaviyo with Shopify. It syncs your store data in real-time, enabling personalized emails based on customer behavior.

---

## Prerequisites

Before starting, you'll need:
- An active Shopify store
- Admin access to your Shopify account
- A business email address (not Gmail/Yahoo for sending)

**Time to complete:** 45-60 minutes

**Cost:** Free for up to 250 email contacts

---

## Table of Contents

1. [Installing Klaviyo on Shopify](#step-1-installing-klaviyo-on-shopify)
2. [Configuring Your Account](#step-2-configuring-your-account)
3. [Syncing Your Shopify Data](#step-3-syncing-your-shopify-data)
4. [Creating Your Email List](#step-4-creating-your-email-list)
5. [Building a Signup Form](#step-5-building-a-signup-form)
6. [Setting Up the Welcome Series Flow](#step-6-setting-up-the-welcome-series-flow)
7. [Creating an Abandoned Cart Flow](#step-7-creating-an-abandoned-cart-flow)
8. [Sending Your First Campaign](#step-8-sending-your-first-campaign)
9. [Understanding Analytics](#step-9-understanding-analytics)
10. [Next Steps](#step-10-next-steps)

---

## Step 1: Installing Klaviyo on Shopify

Let's connect Klaviyo to your Shopify store.

### 1.1 Add Klaviyo from the App Store

1. Log in to your **Shopify Admin**
2. Click **Apps** in the left sidebar
3. Click **Shopify App Store**
4. Search for **"Klaviyo"**
5. Select **"Klaviyo: Email Marketing & SMS"**

> ⚠️ **Important:** Choose "Klaviyo: Email Marketing & SMS" — not "Klaviyo Reviews" (that's a separate product).

### 1.2 Install the App

1. Click **Install**
2. Review the permissions Klaviyo requires:
   - Read products, customers, and orders
   - Write customer tags and marketing preferences
3. Click **Install app**

### 1.3 Create Your Klaviyo Account

After installation, you'll be prompted to create an account:

1. Enter your **email address**
2. Create a **password**
3. Fill in your **company name**
4. Enter your **website URL**
5. Click **Create Account**

If you already have a Klaviyo account, click **Sign in** instead.

---

## Step 2: Configuring Your Account

Now let's set up your sender information.

### 2.1 Set Your Sender Details

Klaviyo will guide you through initial setup. Configure these settings:

**Organization Name:**
```
Your Store Name
```

**Default Sender Email:**
```
hello@yourdomain.com
```

**Default Sender Name:**
```
Your Store Name
```

> **Best Practice:** Use a branded email (hello@yourdomain.com) rather than a personal one. This improves deliverability and looks more professional.

### 2.2 Add Your Physical Address

Email laws require a physical mailing address in every email:

1. Go to **Settings** → **Account** → **Organization**
2. Enter your business address
3. Click **Save**

This address appears in your email footer automatically.

### 2.3 Authenticate Your Domain (Recommended)

Domain authentication improves email deliverability:

1. Go to **Settings** → **Domains**
2. Click **Add Domain**
3. Enter your domain (e.g., `yourdomain.com`)
4. Add the DNS records Klaviyo provides to your domain registrar
5. Click **Verify**

> **Note:** DNS changes can take up to 48 hours to propagate.

---

## Step 3: Syncing Your Shopify Data

Klaviyo automatically syncs data from Shopify. Let's verify the connection.

### 3.1 Check Integration Settings

1. Go to **Integrations** in the left sidebar
2. Find **Shopify** and click on it
3. Verify these settings:

**Sync Settings Checklist:**

| Setting | Recommended |
|---------|-------------|
| Sync Shopify email subscribers | ✅ Enabled |
| Sync Shopify SMS subscribers | ✅ Enabled (if using SMS) |
| Sync profiles to Shopify | ✅ Enabled |
| Enable on-site tracking | ✅ Enabled |

### 3.2 Understanding What Syncs

Klaviyo automatically imports:

| Data Type | What's Included |
|-----------|-----------------|
| **Customers** | Names, emails, phone numbers |
| **Orders** | Purchase history, order values, products bought |
| **Products** | Catalog with images, prices, descriptions |
| **Behavior** | Page views, cart activity, checkout starts |

This data powers personalized emails and segmentation.

### 3.3 Verify Data is Syncing

1. Go to **Audience** → **Profiles**
2. You should see your existing customers listed
3. Click on any profile to see their complete history

```
Profile: john@example.com
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📧 Email: john@example.com
📱 Phone: +234 801 234 5678
💰 Lifetime Value: ₦125,000
🛒 Orders: 3
📅 Last Order: January 1, 2026
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Step 4: Creating Your Email List

Lists organize your subscribers. Let's create your main list.

### 4.1 Create a Newsletter List

1. Go to **Audience** → **Lists & Segments**
2. Click **Create List/Segment**
3. Select **List**
4. Name it: **"Newsletter Subscribers"**
5. Click **Create List**

### 4.2 Understanding Lists vs Segments

| Feature | Lists | Segments |
|---------|-------|----------|
| **How people join** | Manually or via forms | Automatically by meeting conditions |
| **Updates** | Static (people stay until removed) | Dynamic (updates in real-time) |
| **Use case** | Newsletter signups | Targeting based on behavior |

**Example segments you'll create later:**
- Customers who haven't purchased in 90 days
- VIP customers (spent over ₦100,000)
- People who viewed a product but didn't buy

---

## Step 5: Building a Signup Form

Capture email addresses with an embedded or popup form.

### 5.1 Create a Popup Form

1. Go to **Signup Forms** in the left sidebar
2. Click **Create Signup Form**
3. Select **Popup**
4. Choose a template or start from scratch

### 5.2 Design Your Form

Customize these elements:

**Headline:**
```
Get 10% Off Your First Order
```

**Subheadline:**
```
Join our newsletter for exclusive deals and new arrivals.
```

**Form Fields:**
- Email (required)
- First Name (optional but recommended)

**Button Text:**
```
Get My Discount
```

### 5.3 Configure Form Behavior

Click on **Behaviors** tab:

| Setting | Recommended Value |
|---------|-------------------|
| Show after | 5 seconds |
| Show on | All pages |
| Don't show again for | 30 days (after close) |
| Hide on mobile | No (keep enabled) |

### 5.4 Set the Success Action

When someone submits the form:

1. Go to **Success** tab
2. Select **Show a success message**
3. Enter message:
```
Thanks for subscribing! Check your email for your 10% discount code.
```

### 5.5 Connect to Your List

1. Click **Publishing** tab
2. Under **Add to list**, select **Newsletter Subscribers**
3. Click **Publish**

### 5.6 Enable on Your Store

1. Go to **Targeting** tab
2. Ensure your Shopify store is selected
3. Toggle the form to **Live**

---

## Step 6: Setting Up the Welcome Series Flow

A welcome series automatically emails new subscribers. This is your highest-ROI automation.

### 6.1 Access Flow Templates

1. Go to **Flows** in the left sidebar
2. Click **Create Flow**
3. Search for **"Welcome Series"**
4. Select a template and click **Use This Flow**

### 6.2 Customize the Trigger

The flow should trigger when someone joins your newsletter list:

1. Click on the **Trigger** block
2. Ensure it says: **"When someone subscribes to Newsletter Subscribers"**

### 6.3 Edit Email 1: Welcome + Discount

Click on the first email block to edit:

**Subject Line:**
```
Welcome! Here's your 10% discount 🎉
```

**Preview Text:**
```
Your exclusive code is inside...
```

**Email Content Structure:**

```
[Header Image: Your Logo or Brand Image]

Hi {{ first_name|default:"there" }},

Welcome to [Your Store Name]! 

We're thrilled to have you join our community of [X] happy customers.

As promised, here's your exclusive discount:

[BUTTON: Use Code WELCOME10]

This code gives you 10% off your entire order. 
It expires in 7 days, so don't wait!

[FEATURED PRODUCTS SECTION]
Here are some customer favorites to get you started:

[Product Block - pulls from your Shopify catalog]

Questions? Reply to this email – we read every message.

Cheers,
[Your Name]
[Your Store Name]

[Footer with social links and unsubscribe]
```

### 6.4 Add Time Delays

Between emails, add delays:

1. After Email 1: **2-day delay**
2. After Email 2: **3-day delay**

To add a delay:
1. Click the **+** between email blocks
2. Select **Time Delay**
3. Set to **2 days**

### 6.5 Edit Email 2: Brand Story

**Subject Line:**
```
The story behind [Your Store Name]
```

**Content Focus:**
- Share your brand story
- Explain what makes you different
- Include customer testimonials
- Soft product recommendations

### 6.6 Edit Email 3: Social Proof

**Subject Line:**
```
See what our customers are saying
```

**Content Focus:**
- Customer reviews and photos
- User-generated content
- Best-selling products
- Clear call-to-action to shop

### 6.7 Activate the Flow

1. Review all emails
2. Click **Review and Turn On** (top right)
3. Confirm by clicking **Turn On**

Your welcome series is now live!

---

## Step 7: Creating an Abandoned Cart Flow

Recover lost sales by emailing customers who add items to cart but don't checkout.

### 7.1 Create from Template

1. Go to **Flows** → **Create Flow**
2. Search for **"Abandoned Cart"**
3. Select **"Abandoned Cart Reminder"**
4. Click **Use This Flow**

### 7.2 Understanding the Trigger

The flow triggers when:
- Someone adds a product to their cart
- They don't complete checkout within a set time
- They have a known email address

### 7.3 Configure Timing

**Recommended delays:**

| Email | Delay After Cart Abandonment |
|-------|------------------------------|
| Email 1 | 4 hours |
| Email 2 | 24 hours |
| Email 3 | 3 days |

### 7.4 Edit Email 1: Gentle Reminder

**Subject Line Options:**
```
Did you forget something?
You left items in your cart 🛒
Still thinking it over?
```

**Email Content:**

```
Hi {{ first_name|default:"there" }},

Looks like you left some items behind!

[DYNAMIC CART ITEMS BLOCK - shows their actual cart]

Your cart is saved, but these items are popular 
and might sell out.

[BUTTON: Complete My Order]

Need help? Just reply to this email.

[Your Store Name]
```

### 7.5 Add Dynamic Cart Content

Klaviyo can automatically show the exact products in their cart:

1. In the email editor, click **+** to add a block
2. Select **Dynamic** → **Table**
3. Choose **Abandoned Cart Items**

This pulls:
- Product image
- Product name
- Price
- Quantity

### 7.6 Edit Email 2: Add Urgency

**Subject Line:**
```
Your cart expires soon ⏰
```

**Content additions:**
- Mention limited stock
- Add customer reviews
- Consider a small discount (optional)

### 7.7 Edit Email 3: Final Reminder + Incentive

**Subject Line:**
```
Last chance: 10% off to complete your order
```

**Content:**
- Offer a discount code
- Create urgency (cart expires)
- Make checkout extremely easy

### 7.8 Add Filters (Important!)

Prevent emails from sending to people who already purchased:

1. Click on the first email in the flow
2. Click **Add Conditional Split** above it
3. Set condition: **"Has placed order since starting this flow"**
4. Send "Yes" path to exit (they purchased)
5. Send "No" path to continue receiving emails

### 7.9 Activate the Flow

1. Click **Review and Turn On**
2. Confirm activation

---

## Step 8: Sending Your First Campaign

Flows are automated. Campaigns are one-time sends.

### 8.1 Create a Campaign

1. Go to **Campaigns** in the left sidebar
2. Click **Create Campaign**
3. Select **Email**
4. Name it: **"January New Arrivals"**

### 8.2 Select Recipients

Choose who receives this email:

1. Click **Add recipients**
2. Select your **Newsletter Subscribers** list
3. Optionally exclude recent purchasers or other segments

**Smart Sending:** Enable this to skip people who received an email in the last 16 hours.

### 8.3 Design Your Email

1. Click **Continue to Content**
2. Choose a template or start blank
3. Design your email with:

**Essential Elements:**
- Eye-catching header image
- Clear headline
- 1-2 featured products or offers
- Single, prominent call-to-action button
- Mobile-responsive design

### 8.4 Write Compelling Copy

**Subject Line Tips:**

| Type | Example |
|------|---------|
| Curiosity | "You won't believe what just arrived..." |
| Urgency | "24 hours only: Free shipping" |
| Personal | "{{ first_name }}, we picked these for you" |
| Benefit | "Save 3 hours a week with this" |

**Preview Text:**
Use this to complement your subject line, not repeat it.

### 8.5 Preview and Test

1. Click **Preview** to see desktop and mobile views
2. Click **Send Test** to email yourself
3. Check all links work
4. Verify images load correctly
5. Test on mobile

### 8.6 Schedule or Send

**Option 1: Send Now**
1. Click **Review and Send**
2. Confirm recipients and content
3. Click **Send**

**Option 2: Schedule**
1. Click **Schedule**
2. Choose date and time
3. Consider your audience's timezone
4. Click **Schedule Campaign**

**Best send times (general guidance):**
- Tuesday-Thursday
- 10 AM or 2 PM local time
- Avoid Mondays and weekends

---

## Step 9: Understanding Analytics

Track performance to improve over time.

### 9.1 Campaign Analytics

After sending, view results in **Campaigns**:

| Metric | Good Benchmark | What It Means |
|--------|----------------|---------------|
| **Open Rate** | 20-30% | % who opened the email |
| **Click Rate** | 2-5% | % who clicked a link |
| **Conversion Rate** | 1-3% | % who made a purchase |
| **Revenue** | Varies | $ attributed to this email |
| **Unsubscribe Rate** | <0.5% | % who unsubscribed |

### 9.2 Flow Analytics

View flow performance in **Flows** → select a flow → **Analytics**:

**Key metrics:**
- Revenue per recipient
- Total revenue generated
- Email-by-email performance

### 9.3 Reading Customer Profiles

Click on any profile to see:

```
Activity Timeline
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📧 Opened "January New Arrivals"     Today
🛒 Added to cart: Blue Widget        Yesterday
📦 Placed Order #1234                Last week
👋 Subscribed to Newsletter          2 weeks ago
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

This timeline helps you understand customer journeys.

---

## Step 10: Next Steps

You've set up the essentials! Here's what to tackle next:

### Recommended Flows to Add

| Flow | Purpose | Priority |
|------|---------|----------|
| **Browse Abandonment** | Email people who viewed products but didn't add to cart | High |
| **Post-Purchase** | Thank customers, ask for reviews, cross-sell | High |
| **Win-Back** | Re-engage customers who haven't purchased in 90 days | Medium |
| **Birthday** | Send birthday discounts | Medium |
| **VIP** | Reward your best customers | Medium |

### Segmentation Ideas

Create these segments for targeted campaigns:

- **Repeat Customers:** Placed order more than once
- **VIPs:** Total spent > ₦100,000
- **At Risk:** Last purchase 60-90 days ago
- **Window Shoppers:** Viewed products but never purchased
- **By Product Category:** Purchased from specific collection

### A/B Testing

Test these elements to improve performance:

- Subject lines (most impactful)
- Send times
- Email design
- Call-to-action buttons
- Discount amounts

### SMS Marketing

Klaviyo also supports SMS:
- Higher open rates (98% vs 20% for email)
- Best for time-sensitive offers
- Requires separate consent
- Free for first 150 SMS/MMS credits

---

## Quick Reference

### Klaviyo Navigation

| Section | What It Does |
|---------|--------------|
| **Campaigns** | One-time email sends |
| **Flows** | Automated email sequences |
| **Signup Forms** | Pop-ups, embedded forms |
| **Audience** | Lists, segments, profiles |
| **Analytics** | Performance dashboards |
| **Integrations** | Connected apps (Shopify, etc.) |

### Personalization Tags

Use these in subject lines and email content:

```
{{ first_name|default:"there" }}     → "John" or "there"
{{ email }}                           → "john@example.com"
{{ organization.name }}               → "Your Store Name"
```

### Common Troubleshooting

| Problem | Solution |
|---------|----------|
| Emails going to spam | Authenticate your domain |
| Low open rates | Test subject lines, clean your list |
| Flow not triggering | Check trigger conditions and filters |
| Forms not showing | Verify targeting and live status |
| Data not syncing | Check Shopify integration settings |

---

## Summary

You've accomplished a lot! Here's what you set up:

✅ Klaviyo integrated with Shopify
✅ Email list created
✅ Signup form capturing subscribers
✅ Welcome series converting new subscribers
✅ Abandoned cart flow recovering lost sales
✅ First campaign sent

These automations will now work 24/7, generating revenue while you focus on other parts of your business.

---

## Resources

- [Klaviyo Help Center](https://help.klaviyo.com/)
- [Klaviyo Academy](https://academy.klaviyo.com/) (free courses)
- [Shopify Email Marketing Guide](https://www.shopify.com/blog/email-marketing)
- [Klaviyo Community](https://community.klaviyo.com/)

---

*Last updated: January 2026*

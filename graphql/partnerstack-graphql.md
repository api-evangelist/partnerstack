# PartnerStack GraphQL Schema

Conceptual GraphQL schema for the [PartnerStack](https://partnerstack.com/) partner relationship management (PRM) platform. PartnerStack powers partner-led growth programs for B2B SaaS companies, providing programmatic access to partnerships, customers, deals, transactions, and rewards.

- **API Reference**: https://docs.partnerstack.com/reference
- **Documentation**: https://docs.partnerstack.com/
- **Base REST API**: https://api.partnerstack.com/api/v2
- **Schema file**: [partnerstack-schema.graphql](partnerstack-schema.graphql)

## Overview

PartnerStack enables companies to manage affiliate, referral, and reseller partner programs at scale. The GraphQL schema covers the full lifecycle of partner relationship management: onboarding and application review, campaign and offer management, link and coupon tracking, lead and transaction processing, reward calculation, and payout disbursement.

## Schema Summary

The schema defines **70 named types** organized into the following domains:

### Enums (10)

| Enum | Values |
|------|--------|
| `PartnerStatus` | PENDING, APPROVED, REJECTED, SUSPENDED, ACTIVE, INACTIVE |
| `PartnerType` | AFFILIATE, REFERRAL, RESELLER, INFLUENCER, INTEGRATION, TECHNOLOGY, AGENCY |
| `CampaignType` | AFFILIATE, REFERRAL, RESELLER, LEAD_GENERATION, CO_SELL |
| `CampaignGoal` | SIGNUPS, PURCHASES, DEMOS, TRIALS, UPGRADES, CUSTOM |
| `OfferType` | PERCENTAGE_DISCOUNT, FIXED_DISCOUNT, FREE_TRIAL, CREDITS, CUSTOM |
| `CommissionType` | PERCENTAGE, FLAT, TIERED, RECURRING, ONE_TIME |
| `RewardStatus` | PENDING, APPROVED, PAID, CANCELLED, HELD |
| `LeadStatus` | NEW, CONTACTED, QUALIFIED, CONVERTED, DISQUALIFIED, CLOSED_WON, CLOSED_LOST |
| `TransactionStatus` | PENDING, CONFIRMED, CANCELLED, REFUNDED, DISPUTED |
| `PayoutStatus` | PENDING, PROCESSING, COMPLETED, FAILED, CANCELLED, ON_HOLD |
| `PayoutMethod` | PAYPAL, STRIPE, WIRE_TRANSFER, CHECK, CRYPTO, STORE_CREDIT |
| `UserRole` | ADMIN, MANAGER, OPERATOR, VIEWER, PARTNER |
| `ApplicationStatus` | PENDING, APPROVED, REJECTED, WAITLISTED |

### Core Partner Types (5)

- **`Partner`** — Central partner entity with status, type, group membership, and aggregate relationships
- **`PartnerDetails`** — Contact, company, address, payment, and profile metadata
- **`Affiliate`** — Affiliate-specific metrics: click/conversion counts, earnings, links, and coupons
- **`Referral`** — Individual referral record linking a partner to a customer or lead with commission tracking
- **`Reseller`** — Reseller-specific data: tier, discount rate, deal registrations, and custom pricing

### Group Types (3)

- **`GroupPartner`** — Join table connecting partners to groups with role and join date
- **`PartnerGroup`** — Named collection of partners tied to a campaign
- **`GroupDetails`** — Description, logo, website, industry, region, and tier metadata for a group

### Campaign Types (2)

- **`Campaign`** — Program container with type, goal, offers, links, partners, groups, and analytics
- **`CampaignDetails`** — Landing page, terms, banners, email templates, default commission and offer configuration

### Link Types (5)

- **`Link`** — Tracking link with click/conversion metrics
- **`LinkDetails`** — Destination URL, tracking URL, and UTM parameters
- **`LinkAlias`** — Short custom alias for a link
- **`ReferralLink`** — Partner-specific referral link with signup tracking
- **`PartnerLink`** — Assignment of a link to a partner, optionally with a custom alias

### Offer and Commission Types (4)

- **`Offer`** — Discount, trial, or credit offer attached to a campaign
- **`OfferDetails`** — Discount value, trial days, credit amount, expiration, and usage limits
- **`Commission`** — Commission structure (percentage, flat, tiered, recurring)
- **`CommissionRule`** — Individual rule within a tiered or conditional commission structure

### Reward Types (2)

- **`Reward`** — Earned reward record linking a commission, transaction, and payout
- **`RewardDetails`** — Amount, currency, trigger event, approval and payment timestamps

### Coupon Types (3)

- **`Coupon`** — Promo code attached to an offer, partner, and campaign
- **`CouponDetails`** — Discount type/value, usage limits, expiration, and applicable products
- **`CouponUsage`** — Individual redemption record with customer, transaction, and discount applied

### Lead Types (2)

- **`Lead`** — Sales lead generated through a partner referral
- **`LeadDetails`** — Contact info, company, qualification score, and custom fields

### Transaction Types (2)

- **`Transaction`** — Purchase or subscription event attributed to a partner
- **`TransactionDetails`** — Amount, currency, product, subscription, invoice, and metadata

### Payout Types (4)

- **`Payout`** — Disbursement of accumulated rewards to a partner
- **`PayoutDetails`** — Amount, fees, net amount, payment account, and status timestamps
- **`Invoicing`** — Invoice generated for a payout with line items and due date
- **`Billing`** — Partner billing profile: company, address, tax ID, VAT number

### Customer Types (2)

- **`Customer`** — End customer referred or attributed to a partner
- **`CustomerDetails`** — Contact, company, subscription plan, and lifetime value

### Integration Types (4)

- **`Integration`** — Generic third-party integration with configuration and webhooks
- **`SlackIntegration`** — Slack workspace connection with channel and notification event settings
- **`JiraIntegration`** — Jira project connection with issue type mapping and sync state
- **`Webhook`** — Outbound webhook endpoint with event subscription and delivery tracking

### Application Types (2)

- **`Application`** — Partner program application with review status
- **`ApplicationDetails`** — Applicant contact info, marketing channels, audience, and custom fields

### User and Team Types (3)

- **`User`** — Platform user with role, team membership, and API key access
- **`UserDetails`** — Name, email, timezone, language, and notification preferences
- **`Team`** — Collection of users with member count

### Dashboard and Analytics Types (3)

- **`Dashboard`** — Configurable analytics view tied to a campaign or partner
- **`Analytics`** — Aggregated metrics: clicks, conversions, revenue, commissions, top partners/links/coupons, and time-series data
- **`PartnerScore`** — Composite engagement, revenue, activity, and quality score with rank and tier

### Auth Types (3)

- **`APIKey`** — Scoped API key for programmatic access
- **`Token`** — OAuth or session token with scope and expiration
- **`Webhook`** — (see Integration Types above)

## Key Query Operations

```graphql
# Retrieve a partner with their performance data
query GetPartner($id: ID!) {
  partner(id: $id) {
    id
    key
    email
    status
    type
    details {
      firstName
      lastName
      company
    }
    score {
      totalScore
      tier
      rank
    }
    rewards(status: PENDING) {
      id
      details { amount currency }
    }
  }
}

# List partners in a campaign with analytics
query CampaignPartners($campaignId: ID!) {
  campaign(id: $campaignId) {
    name
    type
    partners { id email status }
    analytics {
      clicks
      conversions
      revenue
      commissionsPaid
    }
  }
}

# Get transactions for a time period
query Transactions($partnerId: ID!, $start: DateTime!, $end: DateTime!) {
  transactions(
    partnerId: $partnerId
    startDate: $start
    endDate: $end
    status: CONFIRMED
  ) {
    id
    details { amount currency productName }
    rewards { id status details { amount } }
  }
}
```

## Key Mutation Operations

```graphql
# Approve a pending partner application
mutation ApprovePartner($id: ID!) {
  approvePartner(id: $id) {
    id
    status
    updatedAt
  }
}

# Create a transaction and trigger commission calculation
mutation CreateTransaction($input: TransactionInput!) {
  createTransaction(input: $input) {
    id
    status
    details { amount currency }
    rewards {
      id
      status
      details { amount currency }
    }
  }
}

# Initiate a payout to a partner
mutation CreatePayout($input: PayoutInput!) {
  createPayout(input: $input) {
    id
    status
    method
    details { amount netAmount currency }
  }
}
```

## Notes

- PartnerStack exposes a REST API at `https://api.partnerstack.com/api/v2`. This GraphQL schema is a conceptual representation of the platform's data model derived from the public REST API documentation.
- The REST API uses API key authentication; the `APIKey` and `Token` types in this schema represent those constructs.
- `key` fields throughout the schema correspond to PartnerStack's internal string identifiers used in REST endpoints.
- Custom fields (`customFields: JSON`) are supported on partners, leads, applications, and customers to accommodate per-company data requirements.
- Reward approval and payout processing follow an async workflow; poll `RewardStatus` and `PayoutStatus` for state transitions.

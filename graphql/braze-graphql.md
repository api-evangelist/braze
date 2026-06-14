# Braze GraphQL Schema

This is a conceptual GraphQL schema for the Braze customer engagement platform. Braze exposes a REST API; this GraphQL schema is a normalized representation of the data models, relationships, and operations available through the Braze REST API (https://www.braze.com/docs/api/).

## Overview

Braze is a customer engagement platform that enables brands to deliver personalized, multi-channel messaging across mobile push, web push, email, in-app messages, content cards, SMS, and WhatsApp. The schema below maps the core Braze data model into GraphQL types, covering user profiles, campaigns, canvases, messaging, segmentation, catalogs, and analytics.

## Schema Source

- REST API Docs: https://www.braze.com/docs/api/basics/
- Rate Limits: https://www.braze.com/docs/api/api_limits/
- GitHub: https://github.com/braze-inc
- Authentication: Bearer token (API key per workspace/app group)

## Type Categories

### User & Identity
- `User` — core user profile with external ID, aliases, attributes, and events
- `UserAlias` — alternative identifiers (alias label + name)
- `UserIdentification` — identifier mapping for merging anonymous and known users
- `UserExternalId` — external identifier with rename/merge support
- `UserProfile` — full expanded profile including attributes, purchases, sessions
- `UserAttribute` — key/value custom attribute on a user
- `UserEvent` — custom event logged against a user profile

### Commerce & Sessions
- `Purchase` — logged product purchase with product ID, currency, price, quantity
- `Session` — app session record (start time, duration, platform)

### Subscriptions
- `Subscription` — email or push subscription state for a user
- `SubscriptionGroup` — SMS or email subscription group membership

### Campaigns & Canvases
- `Campaign` — marketing campaign container with variants and schedule
- `CampaignStats` — delivery and engagement metrics for a campaign
- `Canvas` — multi-step journey with branching logic
- `CanvasStep` — individual step in a Canvas (message, delay, decision)
- `CanvasVariant` — A/B variant within a Canvas

### Messaging
- `Message` — generic message container
- `MessageVariant` — versioned message variant for A/B testing
- `MessageSend` — record of a message send action
- `MessageSchedule` — scheduled message delivery configuration
- `PushNotification` — push notification payload (title, body, badge, sound)
- `Email` — email message (subject, from, reply-to, body, attachments)
- `InAppMessage` — in-app message (type, header, body, buttons, display)
- `ContentCard` — persistent content card for the card feed
- `SMSMessage` — SMS body and media (MMS) content
- `WhatsAppMessage` — WhatsApp template-based message
- `Webhook` — outbound webhook call details
- `BrazeWebhook` — Braze-native webhook configuration and history

### Segmentation
- `Segment` — saved audience segment with filter criteria
- `SegmentData` — analytics rollup for a segment (size, trend)
- `SegmentFilter` — individual filter clause within a segment definition

### Content Feeds & Cards
- `Feed` — deprecated News Feed container
- `FeedCard` — individual News Feed card (title, description, image)

### Templates
- `Template` — generic reusable content template
- `EmailTemplate` — full email template (HTML, plaintext, subject)
- `PushTemplate` — push notification template
- `ContentBlock` — reusable content block for drag-and-drop email editor

### Catalogs
- `Catalog` — product or content catalog for personalization
- `CatalogItem` — individual item in a catalog with custom fields
- `CatalogField` — field definition for a catalog schema

### News & Promotions
- `NewsItem` — in-app news item
- `Promotion` — promotional campaign entity
- `PromotionCode` — single-use or multi-use promo code within a promotion

### Data Management
- `DataExport` — export job request and status for user/campaign data
- `DataImport` — bulk import job for user profile data

### Analytics & Dashboards
- `Dashboard` — analytics dashboard reference
- `EventProperty` — typed property definition for a custom event schema

### Custom Data
- `CustomEvent` — custom event type definition registered in the dashboard
- `CustomAttribute` — custom attribute schema definition (type, name)

### Platform & Auth
- `Integration` — third-party integration configuration
- `APIKey` — API key with permissions and workspace scope
- `AppGroup` — workspace (app group) grouping apps and settings
- `App` — individual app (iOS, Android, Web) within a workspace
- `Platform` — platform enum (iOS, Android, Web, Kindle, tvOS)

## Operations

### Queries
- `user(externalId: String!)` — fetch a user profile
- `campaign(campaignId: ID!)` — fetch campaign details
- `campaignStats(campaignId: ID!)` — fetch delivery/engagement metrics
- `canvas(canvasId: ID!)` — fetch canvas structure
- `segment(segmentId: ID!)` — fetch segment definition
- `segments` — list all segments
- `catalog(catalogId: ID!)` — fetch a catalog
- `catalogItems(catalogId: ID!)` — list items in a catalog
- `emailTemplate(templateId: ID!)` — fetch email template
- `contentBlock(blockId: ID!)` — fetch content block
- `subscriptionGroup(groupId: ID!)` — fetch subscription group
- `apiKeys` — list API keys for the workspace
- `appGroup(appGroupId: ID!)` — fetch workspace/app group
- `dataExport(exportId: ID!)` — check data export job status
- `customEvents` — list registered custom event types
- `customAttributes` — list registered custom attribute schemas

### Mutations
- `trackUsers(input: TrackUsersInput!)` — track user attributes, events, purchases
- `deleteUser(externalId: String!)` — delete user data (GDPR)
- `mergeUsers(input: MergeUsersInput!)` — merge two user profiles
- `sendMessage(input: SendMessageInput!)` — send a message immediately
- `scheduleMessage(input: ScheduleMessageInput!)` — schedule a future message
- `triggerCampaign(input: TriggerCampaignInput!)` — API-trigger a campaign
- `triggerCanvas(input: TriggerCanvasInput!)` — API-trigger a canvas
- `createSegment(input: CreateSegmentInput!)` — create a new segment
- `updateSubscriptionStatus(input: UpdateSubscriptionInput!)` — update email/push subscription
- `createCatalogItem(input: CreateCatalogItemInput!)` — add item to catalog
- `updateCatalogItem(input: UpdateCatalogItemInput!)` — update catalog item
- `deleteCatalogItem(catalogId: ID!, itemId: ID!)` — remove catalog item
- `createEmailTemplate(input: CreateEmailTemplateInput!)` — create email template
- `createContentBlock(input: CreateContentBlockInput!)` — create content block
- `requestDataExport(input: DataExportInput!)` — start a data export job

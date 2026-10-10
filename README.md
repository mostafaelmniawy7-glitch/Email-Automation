# AI Email Automation System

Intelligent email assistant that classifies, replies, and manages incoming emails using AI. Built with n8n, OpenAI, and Supabase.

## Overview

An automated email management system that:
- Classifies incoming emails by category, priority, and sentiment
- Auto-replies to questions
- Archives spam
- Extracts leads from potential customers
- Sends Telegram notifications
- Stores everything in Supabase

## Features

### AI Classification
- Category (question / order / complaint / spam / newsletter)
- Priority (high / medium / low)
- Sentiment (positive / neutral / negative / angry)
- Auto-summary

### Automated Actions
- **Auto-Reply** — Sends intelligent replies to questions
- **Spam Archive** — Auto-archives spam emails
- **Lead Extraction** — Extracts contact info and intent from potential customers
- **Human Handoff** — Flags emails needing human attention

### Notifications
- Real-time Telegram alerts
- Priority color coding (🔴 high / 🟡 medium / 🟢 low)
- Lead scoring with dedicated alerts

### Data Persistence
- All emails stored in Supabase
- Lead database with scoring
- Duplicate prevention
- Reply tracking

## System Architecture

## 1. Email Ingestion
Gmail Trigger polls for new unread emails every minute and passes them to the extraction stage.

## 2. Data Extraction
Extracts sender info, subject, body, message ID, and labels from the raw Gmail payload.

## 3. AI Classification
OpenAI GPT-4o-mini analyzes the email and returns structured JSON containing:
- Category (question / order / complaint / spam / newsletter / other)
- Priority (high / medium / low)
- Sentiment (positive / neutral / negative / angry)
- Summary and suggested reply
- Handoff flag
- Lead detection (is_lead, lead_data, lead_score)

## 4. Response Parsing
Normalizes the AI response into a flat object and merges it with the original email metadata.

## 5. Duplicate Prevention
Queries Supabase to check if the email already exists by message_id. Skips processing if found.

## 6. Persistence
Saves the classified email into the bot_emails table in Supabase.

## 7. Parallel Processing (3 Branches)

### Branch A — Lead Management
- Checks if the email is a qualified lead
- If true: saves to bot_leads table and sends a dedicated Telegram alert

### Branch B — Spam Handling
- Checks if the email is spam
- If true: applies the SPAM label via Gmail API (auto-archive)

### Branch C — Auto-Reply
- Checks if the email qualifies for auto-reply (question + no handoff)
- If true: sends an AI-generated reply via Gmail
- Updates the email record with reply status and text

## 8. Notification
Builds a formatted message with priority color coding and sends it to Telegram.

## External Services
- **Gmail API** — Email reading, replying, labeling
- **OpenAI API** — Classification and reply generation
- **Supabase** — PostgreSQL storage (bot_emails, bot_leads)
- **Telegram Bot API** — Real-time notifications

## Data Flow
Gmail Trigger → Extract → AI Classify → Parse → Check Duplicate → Save → Branch (Lead / Spam / Reply) → Notify Telegram

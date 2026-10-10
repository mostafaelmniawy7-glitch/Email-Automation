# AI Email Automation System

> An intelligent email assistant that classifies, replies, and manages incoming emails using AI.

![Status](https://img.shields.io/badge/status-production--ready-brightgreen)
![n8n](https://img.shields.io/badge/n8n-workflow-orange)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-blue)
![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-green)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Database Schema](#database-schema)
- [Setup](#setup)
- [Use Cases](#use-cases)
- [Screenshots](#screenshots)
- [Author](#author)

---

## Overview

An AI-powered email automation system that transforms a standard inbox into an intelligent assistant. Every incoming email is analyzed, classified, and routed automatically — with the right action taken for each type.

Built with **n8n**, **OpenAI GPT-4o-mini**, and **Supabase**, this system handles the full email lifecycle: from ingestion to classification, auto-reply, lead extraction, spam archiving, and real-time notifications.

---

## Features

### 🧠 AI Classification
Every email is analyzed and tagged with:
- **Category** — question, order, complaint, spam, newsletter, other
- **Priority** — high, medium, low
- **Sentiment** — positive, neutral, negative, angry
- **Summary** — 1-2 sentence digest
- **Labels** — auto-generated topic tags

### ⚡ Automated Actions

| Action | Trigger | Result |
|--------|---------|--------|
| **Auto-Reply** | Question + no handoff | AI-generated reply sent via Gmail |
| **Spam Archive** | Category = spam | Auto-labeled and archived |
| **Lead Extraction** | Buying intent detected | Saved to `bot_leads` |
| **Human Handoff** | Angry / high-priority complaint | Flagged for review |

### 🔔 Real-Time Notifications
- Telegram alerts with priority color coding
- Dedicated lead alerts with scoring
- Formatted HTML messages

### 💾 Data Persistence
- Full email history in Supabase
- Lead database with scoring
- Duplicate prevention by message ID
- Reply tracking per email

---

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

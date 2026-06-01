# OpenSpectrum - Technical Architecture and Technology Stack

## Project Overview

OpenSpectrum is a mobile-first application designed for caregivers and families supporting neurodivergent children.

Primary goals:

* Offline-first operation
* Family collaboration
* Multi-device synchronization
* Secure handling of sensitive data
* AI-assisted guidance and support
* Cross-platform mobile support

---

# Final Technology Stack

## Mobile Frontend

Framework:

* React Native

Language:

* TypeScript

Reasoning:

* Existing team familiarity
* Large ecosystem
* Faster development
* Easier hiring
* Mature community support

---

# Local Data Storage

Database:

* SQLite

Encryption:

* SQLCipher

Purpose:

* Offline-first operation
* Fast local reads and writes
* Data available without internet access

Notes:

* SQLite is the local working database.
* Users primarily interact with local data.
* Synchronization occurs in the background.

---

# Cloud Backend

Preferred Platform:

* Supabase

Services Used:

* Authentication
* PostgreSQL database
* Row Level Security (RLS)

Purpose:

* Account management
* Family sharing
* Device restoration
* Synchronization

---

# Alternative to Supabase

If additional Supabase projects become a limitation:

Preferred alternatives:

1. Self-hosted Supabase
2. PostgreSQL on Railway
3. PostgreSQL on Neon
4. PostgreSQL on Render

Recommended order:

1. Supabase
2. Neon
3. Railway

The application architecture should remain PostgreSQL-compatible to allow migration.

---

# Authentication

Authentication Provider:

* Supabase Auth

Supported Methods:

* Email/password
* Google login

Future Options:

* Apple login
* Microsoft login

---

# Backend API Layer

Framework:

* FastAPI

Language:

* Python

Responsibilities:

* Business logic
* Permission enforcement
* Synchronization services
* AI orchestration
* Reporting
* Analytics

The mobile application should never communicate directly with AI providers.

All AI requests should pass through FastAPI.

---

# AI Layer

Provider:

* NVIDIA NIM API

Purpose:

* Conversational support
* Guidance generation
* Summarization
* Insight generation

Architecture:

React Native App
->
FastAPI Backend
->
NVIDIA NIM API

Benefits:

* Centralized prompt management
* Easier provider replacement
* Better security
* Reduced client complexity

---

# Data Synchronization Model

Architecture Style:

Offline First

Flow:

User
->
SQLite (local)
->
Sync Service
->
PostgreSQL (cloud)

Key Principles:

* Local database is always available.
* App functions without internet.
* Sync runs opportunistically.
* Cloud acts as shared source of truth.

---

# Security Model

Data In Transit

* HTTPS/TLS
* Encrypted API communication

Data On Device

* SQLite
* SQLCipher encryption

Secrets

* iOS Keychain
* Android Keystore

Authentication

* JWT tokens managed by Supabase

---

# Core Domain Tables

users

families

family_members

children

observations

activities

journal_entries

attachments

reports

observation_access

---

# Authorization Model

Administrative Roles:

* Owner
* Editor
* Viewer

Visibility Levels:

* Family
* Parents
* Clinical
* Private
* Custom

Authorization decisions should be enforced by:

1. PostgreSQL RLS policies
2. FastAPI business logic

Never rely solely on client-side checks.

---

# Deployment Architecture

Mobile

* React Native application

Backend

* FastAPI

Database

* PostgreSQL

Authentication

* Supabase Auth

AI

* NVIDIA NIM API

Storage

* Local SQLite
* Cloud PostgreSQL

---

# MVP Scope

Included

* User authentication
* Family management
* Child profiles
* Notes and observations
* Visibility controls
* Offline support
* Cloud synchronization
* AI-powered assistance

Deferred

* End-to-end encryption
* Hospital integrations
* Insurance integrations
* Advanced clinical workflows
* Enterprise audit systems

---

# Guiding Principles

1. Mobile-first.
2. Offline-first.
3. Keep permissions simple.
4. Keep visibility flexible.
5. Store locally, sync to cloud.
6. Centralize AI access through FastAPI.
7. Optimize for caregiver usability rather than enterprise healthcare workflows.
8. Build a maintainable MVP before adding advanced compliance features.

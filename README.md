# Community Alert (working title: melttheice)

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/asciinything/melttheice/ci.yml?branch=main&style=for-the-badge)
![GitHub](https://img.shields.io/github/license/asciinything/melttheice?style=for-the-badge)
![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg?style=for-the-badge)

A self-hosted, open-source application for sharing real-time, location-based information to empower and protect communities.

## 📖 Mission & Guiding Principles

The mission of **Community Alert** is to empower communities by sharing real-time, location-based information about immigration-enforcement activity so individuals can make informed safety decisions.

Our development is guided by these core, non-negotiable principles:

*   🔒 **User Anonymity & Privacy-by-Design:** We collect the absolute minimum data required. We never use traditional accounts, and all sensitive data (like location) is encrypted at rest and automatically deleted after 24 hours.
*   🤝 **Open Source & Community Collaboration:** All core code is MIT licensed. We believe in transparent governance and building a broad, inclusive contributor base.
*   ራ **Operational Independence:** We self-host our infrastructure to ensure predictable costs, avoid vendor lock-in, and maintain full control over user data.
*   ⚖️ **Legal & Ethical Compliance:** We are committed to responsible operation, with clear moderation policies to prevent abuse and a commitment to transparency.

## ✨ Core Features

*   **Anonymous Reporting:** Users can anonymously report a sighting at a specific location without creating an account.
*   **Community Confirmation:** A Waze-like voting system allows other users to confirm or deny a report, building a `credibility_score`.
*   **Proximity-Based Alerts:** Once a report reaches a credibility threshold, anonymous push notifications are sent to users within a defined radius.
*   **Automatic Data Expiration:** To protect privacy, all report and location data is permanently deleted from the database after 24 hours.

## 🛠️ Technology Stack

This project is a monorepo containing the mobile app, backend API, and infrastructure code.

*   **Frontend (Mobile App):**
    *   Framework: **React Native 0.73 (Bare)**
    *   Key Libraries: React Native Maps, Geolocation, CodePush

*   **Backend Services:**
    *   API: **ASP.NET Core 8** (in Docker)
    *   Queue: **Redis Streams** for decoupling report ingestion from push dispatch
    *   Push Workers: Custom C# service for APNS & FCM

*   **Data Layer:**
    *   Database: **PostgreSQL 16** with **PostGIS** extension
    *   Backups: Nightly `pg_dump` and WAL streaming to Backblaze B2

*   **Infrastructure & Hosting:**
    *   Provider: **Hetzner Cloud**
    *   Infrastructure as Code: **Terraform** & **Ansible**
    *   Containerization: **Docker**
    *   Observability: **Prometheus**, **Grafana**, **Uptime Kuma**

## 🚀 Getting Started (For Developers)

Welcome, contributor! Here’s how to get the project running locally.

### Prerequisites

*   [Git](https://git-scm.com/)
*   [Node.js](https://nodejs.org/) (v18+ recommended) and Yarn
*   [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
*   [Docker](https://www.docker.com/) and Docker Compose

### 1. Clone the Repository

```bash
git clone https://github.com/asciinything/melttheice.git
cd melttheice
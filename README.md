<div align="center">

# 🔐 GateVault

### Secure Student Gate Pass Management System

[![Live Demo](https://img.shields.io/badge/Live%20Demo-gatevault--agps.vercel.app-blue?style=for-the-badge&logo=vercel)](https://gatevault-agps.vercel.app/)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-green?style=for-the-badge&logo=mongodb)](https://www.mongodb.com/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-v4-38bdf8?style=for-the-badge&logo=tailwindcss)](https://tailwindcss.com/)
[![Deployed on Vercel](https://img.shields.io/badge/Deployed%20on-Vercel-black?style=for-the-badge&logo=vercel)](https://vercel.com/)

**GateVault** is a full-stack web application designed to digitize and secure the gate pass workflow for universities and educational institutions. It replaces traditional paper-based gate pass systems with a streamlined, QR-code-powered platform — enabling students to request passes digitally, administrators to approve them, and security personnel to verify entry/exit in real time.

[**View Live Demo →**](https://gatevault-agps.vercel.app/)&nbsp;&nbsp;|&nbsp;&nbsp;[**GitHub Repository →**](https://github.com/anandasaikiacse/gatevault)

</div>

---

## 📸 Screenshots

> _Add screenshots of the dashboard, gate pass request flow, and QR scanner here._  
> _(Tip: Use [Scribe](https://scribehow.com/) or browser DevTools to capture clean UI screenshots.)_

---

## ✨ Features

- **Secure Authentication** — Supports both Google OAuth and email/password sign-in via NextAuth.js, with bcrypt-hashed credentials.
- **Gate Pass Requests** — Students can raise and track gate pass requests through a clean, intuitive interface.
- **QR Code Generation** — Approved passes are instantly converted into scannable QR codes for contactless verification.
- **QR Code Scanning** — Security staff can scan passes directly from the browser using the built-in QR scanner (no native app required).
- **Admin Dashboard** — Administrators can review, approve, or reject pass requests with full visibility into pass history.
- **Analytics & Reports** — Visual charts powered by Recharts provide actionable insights into pass usage and trends.
- **Smooth UI Animations** — Framer Motion delivers polished transitions for a professional user experience.
- **Fully Responsive** — Mobile-first design built with Tailwind CSS v4.

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| **Framework** | Next.js 16 (App Router) |
| **Language** | TypeScript 5 |
| **Database** | MongoDB via Mongoose 9 |
| **Authentication** | NextAuth.js v4 (Google OAuth + Credentials) |
| **Styling** | Tailwind CSS v4 |
| **Animations** | Framer Motion |
| **QR Code** | qrcode.react (generation), html5-qrcode (scanning) |
| **Charts** | Recharts |
| **Icons** | Lucide React |
| **Date Handling** | Day.js |
| **Security** | bcryptjs |
| **Deployment** | Vercel |

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed before proceeding:

- **Node.js** v18 or higher
- **npm**, **yarn**, **pnpm**, or **bun**
- A **MongoDB** database (local or [MongoDB Atlas](https://www.mongodb.com/cloud/atlas))
- A **Google OAuth** app (for social login) — [Create one here](https://console.developers.google.com/)

### Installation

**1. Clone the repository:**

```bash
git clone https://github.com/anandasaikiacse/gatevault.git
cd gatevault
```

**2. Install dependencies:**

```bash
npm install
# or
yarn install
# or
pnpm install
```

**3. Configure environment variables:**

Create a `.env.local` file in the root directory and add the following:

```env
# MongoDB
MONGODB_URI=your_mongodb_connection_string

# NextAuth
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:3000

# Google OAuth
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
```

> **Tip:** Generate a strong `NEXTAUTH_SECRET` by running `openssl rand -base64 32` in your terminal.

**4. Start the development server:**

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser to see the application.

---

## 📁 Project Structure

```
gatevault/
├── app/              # Next.js App Router — pages, layouts, and API routes
├── components/       # Reusable React components
├── hooks/            # Custom React hooks
├── lib/              # Utility functions and database connection helpers
├── models/           # Mongoose data models (User, GatePass, etc.)
├── public/           # Static assets
├── types/            # TypeScript type definitions
├── next.config.ts    # Next.js configuration
├── tailwind.config.* # Tailwind CSS configuration
└── tsconfig.json     # TypeScript configuration
```

---

## 🏗️ Architecture Overview

GateVault follows the **Next.js App Router** paradigm with a clear separation of concerns:

- **Frontend** — Server and client components built with React 19, styled with Tailwind CSS v4, and enhanced with Framer Motion animations.
- **Backend** — Next.js API routes handle all server-side logic, including authentication flows, pass management, and database operations.
- **Database Layer** — MongoDB with Mongoose provides a flexible, schema-validated data store for users, gate passes, and audit logs.
- **Authentication** — NextAuth.js manages session handling, JWT tokens, and multi-provider login (Google + credentials).

---

## 📊 Use Cases

GateVault is purpose-built for educational institutions managing student movement. Typical workflows include:

- A **student** logs in, submits a gate pass request specifying the reason, destination, and expected return time.
- An **administrator or warden** reviews pending requests and approves or rejects them with optional remarks.
- Upon approval, the student receives a **QR-coded gate pass** accessible from their dashboard.
- A **security officer** scans the QR code at the gate, instantly verifying pass authenticity and validity without manual record-keeping.

---

## 🌐 Deployment

This project is deployed on **Vercel** and benefits from its global edge network for fast, reliable delivery.

To deploy your own instance:

1. Push your repository to GitHub.
2. Import the project on [Vercel](https://vercel.com/new).
3. Add all environment variables from your `.env.local` under the **Environment Variables** section in the Vercel project settings.
4. Deploy — Vercel handles the build and hosting automatically.

---

## 🔧 Available Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start the development server |
| `npm run build` | Build the application for production |
| `npm run start` | Start the production server |
| `npm run lint` | Run ESLint for code quality checks |

---

## 🤝 Contributing

Contributions, bug reports, and feature suggestions are welcome. To contribute:

1. Fork the repository.
2. Create a new branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m 'feat: add your feature'`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a Pull Request.

Please ensure your code passes `npm run lint` before submitting a PR.

---

## 📄 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Ananda Saikia**

[![GitHub](https://img.shields.io/badge/GitHub-anandasaikiacse-181717?style=flat-square&logo=github)](https://github.com/anandasaikiacse)

---

<div align="center">

⭐ If you found this project useful, consider giving it a star on GitHub — it helps others discover it!

</div>

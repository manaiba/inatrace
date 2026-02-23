<div align="center">

<img src="https://raw.githubusercontent.com/agstack/inatrace/main/.github/inatrace-logo.png" alt="INATrace" width="200" onerror="this.style.display='none'"/>

# INATrace

### Open-Source Blockchain-Based Track & Trace for Agricultural Supply Chains

[![Backend](https://img.shields.io/badge/Backend-Java_17-orange?style=for-the-badge&logo=openjdk)](https://github.com/agstack/inatrace-backend)
[![Frontend](https://img.shields.io/badge/Frontend-Angular-red?style=for-the-badge&logo=angular)](https://github.com/agstack/inatrace-frontend)
[![Mobile](https://img.shields.io/badge/Mobile-React_Native-blue?style=for-the-badge&logo=react)](https://github.com/agstack/inatrace-mobile)
[![Blockchain](https://img.shields.io/badge/Blockchain-Hyperledger_Fabric-2F3134?style=for-the-badge&logo=hyperledger)](https://github.com/agstack/inatrace-coffee-network)

[![Google Play](https://img.shields.io/badge/Google_Play-Download-green?style=flat-square&logo=google-play)](https://play.google.com/store/apps/details?hl=en-US&id=com.sunesis.inatrace)
[![App Store](https://img.shields.io/badge/App_Store-Download-blue?style=flat-square&logo=apple)](https://apps.apple.com/us/app/inatrace/id6572305162)
[![License: MPL-2.0](https://img.shields.io/badge/License-MPL_2.0-brightgreen?style=flat-square)](LICENSE)
[![Part of AgStack](https://img.shields.io/badge/Part_of-AgStack_Foundation-2ea44f?style=flat-square)](https://github.com/agstack)

*An open-source, blockchain-backed system that digitizes agricultural supply chains — connecting farmers, cooperatives, processors, and buyers with full transparency, quality assurance, and fair pricing. Funded by BMZ and implemented by GIZ.*

</div>

---

## 🌍 What is INATrace?

**INATrace** is an end-to-end digital traceability platform for agricultural commodities (coffee, cocoa, and beyond). It provides:

- 🔗 **Full supply chain transparency** — trace every step from smallholder farm to final buyer
- ⛓️ **Blockchain-backed trust** — immutable records on Hyperledger Fabric
- 🏢 **Multi-tenant, multi-value-chain** — one system for multiple organizations and commodity types
- 📱 **Mobile-first field data** — GPS polygon mapping, offline-capable farmer registration
- 📊 **Quality & compliance** — assure quality standards and support EU Deforestation Regulation (EUDR) compliance
- 💰 **Fair pricing** — transparent pricing and payment tracking for smallholder farmers

> *INATrace enhances the economic conditions of smallholder farmers by improving traceability of global supply chains.*

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         INATrace Platform                          │
├──────────────┬──────────────┬─────────────────┬────────────────────┤
│              │              │                 │                    │
│  📱 Mobile   │  🖥️ Frontend  │  ⚙️ Backend      │  ⛓️ Blockchain     │
│  App         │  (Angular)   │  (Java/Spring)  │  (Hyperledger)    │
│              │              │                 │                    │
│ React Native │ TypeScript   │ Java 17         │ Fabric Network    │
│ Expo         │ Angular 10+  │ Spring Boot     │ Chaincode (Go)    │
│ MapBox       │              │ MySQL 8.4       │ CouchDB            │
│              │              │ REST API        │                    │
├──────────────┴──────────────┴─────────────────┴────────────────────┤
│                                                                     │
│  🗄️ MySQL Database    📧 Email (SMTP)    🗺️ MapBox    💱 Exchange API  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📦 Repositories

This is the **meta repository** for the INATrace project. The source code lives in four dedicated repositories:

### ⚙️ [inatrace-backend](https://github.com/agstack/inatrace-backend) — Java API Server

![Stars](https://img.shields.io/github/stars/agstack/inatrace-backend?style=flat-square)
![Forks](https://img.shields.io/github/forks/agstack/inatrace-backend?style=flat-square)
![Last Commit](https://img.shields.io/github/last-commit/agstack/inatrace-backend?style=flat-square)
![Language](https://img.shields.io/github/languages/top/agstack/inatrace-backend?style=flat-square)

The core API server powering INATrace. Handles user management, product configuration, supply chain operations, processing actions, payments, and integrations.

| | |
|---|---|
| **Stack** | Java 17, Spring Boot, Maven, MySQL 8.4 |
| **License** | MPL-2.0 |
| **Last Updated** | 2025-12-24 |
| **Top Maintainers** | [@pece-a](https://github.com/pece-a) (495 commits), [@RebernikNejc](https://github.com/RebernikNejc) (126), [@MusicDin](https://github.com/MusicDin) (96) |

**Quick Start:**
```bash
git clone https://github.com/agstack/inatrace-backend.git
# Set up MySQL via Docker:
docker run --name inatrace-mysql -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=inatrace -e MYSQL_USER=inatrace \
  -e MYSQL_PASSWORD=inatrace -p 3306:3306 -d mysql:8.4.6
# Configure application.properties, then run INATraceBackendApplication.java
```

---

### 🖥️ [inatrace-frontend](https://github.com/agstack/inatrace-frontend) — Angular Web Application

![Stars](https://img.shields.io/github/stars/agstack/inatrace-frontend?style=flat-square)
![Forks](https://img.shields.io/github/forks/agstack/inatrace-frontend?style=flat-square)
![Last Commit](https://img.shields.io/github/last-commit/agstack/inatrace-frontend?style=flat-square)
![Language](https://img.shields.io/github/languages/top/agstack/inatrace-frontend?style=flat-square)

The web-based user interface for managing products, companies, value chains, stakeholders, processing, orders, and payments.

| | |
|---|---|
| **Stack** | Angular 10+, TypeScript, Node 14 |
| **License** | MPL-2.0 |
| **Last Updated** | 2025-12-24 |
| **Top Maintainers** | [@pece-a](https://github.com/pece-a) (987 commits), [@RebernikNejc](https://github.com/RebernikNejc) (175), [@borce-paspalovski](https://github.com/borce-paspalovski) (145) |

**Quick Start:**
```bash
git clone https://github.com/agstack/inatrace-frontend.git
cd inatrace-frontend
npm install
# Configure environment, then:
ng serve
```

---

### 📱 [inatrace-mobile](https://github.com/agstack/inatrace-mobile) — React Native Mobile App

![Stars](https://img.shields.io/github/stars/agstack/inatrace-mobile?style=flat-square)
![Forks](https://img.shields.io/github/forks/agstack/inatrace-mobile?style=flat-square)
![Last Commit](https://img.shields.io/github/last-commit/agstack/inatrace-mobile?style=flat-square)
![Language](https://img.shields.io/github/languages/top/agstack/inatrace-mobile?style=flat-square)

The field-level mobile app for cooperatives and farmers — GPS polygon mapping, farmer registration, offline data collection, and synchronization.

| | |
|---|---|
| **Stack** | React Native, Expo, TypeScript, MapBox |
| **Platforms** | Android ([Google Play](https://play.google.com/store/apps/details?hl=en-US&id=com.sunesis.inatrace)) · iOS ([App Store](https://apps.apple.com/us/app/inatrace/id6572305162)) |
| **License** | MPL-2.0 |
| **Last Updated** | 2025-12-24 |
| **Top Maintainers** | [@pinokokol](https://github.com/pinokokol) (120 commits), [@pece-a](https://github.com/pece-a) (9) |

**Key Features:**
- 📍 **Polygon Mapping** — GPS-based field boundary capture
- 👨‍🌾 **Farmer Profiles** — Detailed cooperative member records
- 📴 **Offline Mode** — Full functionality without connectivity, auto-sync when online

**Quick Start:**
```bash
git clone https://github.com/agstack/inatrace-mobile.git
cd inatrace-mobile
npm install
# Configure .env (API URIs, MapBox tokens)
npx expo run:android   # or run:ios
```

---

### ⛓️ [inatrace-coffee-network](https://github.com/agstack/inatrace-coffee-network) — Hyperledger Fabric Blockchain

![Stars](https://img.shields.io/github/stars/agstack/inatrace-coffee-network?style=flat-square)
![Forks](https://img.shields.io/github/forks/agstack/inatrace-coffee-network?style=flat-square)
![Last Commit](https://img.shields.io/github/last-commit/agstack/inatrace-coffee-network?style=flat-square)
![Language](https://img.shields.io/github/languages/top/agstack/inatrace-coffee-network?style=flat-square)

The blockchain layer — a Hyperledger Fabric network with smart contracts (chaincode) that provide immutable, auditable records for supply chain transactions.

| | |
|---|---|
| **Stack** | Hyperledger Fabric, Docker, CouchDB, Go chaincode |
| **Based On** | [BasicNetwork-2.0](https://github.com/adhavpavan/BasicNetwork-2.0.git), [Fabric Samples](https://github.com/hyperledger/fabric-samples) |
| **License** | MPL-2.0 |
| **Last Updated** | 2025-11-20 |
| **Documentation** | [Hyperledger Fabric Docs](https://hyperledger-fabric.readthedocs.io/) |

---

## 📊 Project Stats

| Metric | Backend | Frontend | Mobile | Blockchain | **Total** |
|--------|---------|----------|--------|------------|-----------|
| ⭐ Stars | 8 | 1 | 0 | 1 | **10** |
| 🍴 Forks | 6 | 5 | 2 | 3 | **16** |
| 🐛 Open Issues | 4 | 0 | 0 | 0 | **4** |
| 📅 Created | 2020-11 | 2020-11 | 2024-12 | 2020-11 | — |
| 📝 Last Push | 2025-12-24 | 2025-12-24 | 2025-12-24 | 2025-11-20 | — |

---

## 👥 Core Maintainers

<table>
<tr>
<td align="center">
<a href="https://github.com/pece-a"><img src="https://github.com/pece-a.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@pece-a</b></sub></a><br/>
<sub>1,492 commits · Backend & Frontend lead</sub>
</td>
<td align="center">
<a href="https://github.com/RebernikNejc"><img src="https://github.com/RebernikNejc.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@RebernikNejc</b></sub></a><br/>
<sub>301 commits · Backend & Frontend</sub>
</td>
<td align="center">
<a href="https://github.com/borce-paspalovski"><img src="https://github.com/borce-paspalovski.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@borce-paspalovski</b></sub></a><br/>
<sub>223 commits · Frontend</sub>
</td>
<td align="center">
<a href="https://github.com/pinokokol"><img src="https://github.com/pinokokol.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@pinokokol</b></sub></a><br/>
<sub>120 commits · Mobile lead</sub>
</td>
<td align="center">
<a href="https://github.com/MusicDin"><img src="https://github.com/MusicDin.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@MusicDin</b></sub></a><br/>
<sub>148 commits · Backend</sub>
</td>
</tr>
<tr>
<td align="center">
<a href="https://github.com/MHA10"><img src="https://github.com/MHA10.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@MHA10</b></sub></a><br/>
<sub>186 commits · Backend</sub>
</td>
<td align="center">
<a href="https://github.com/dockerymick"><img src="https://github.com/dockerymick.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@dockerymick</b></sub></a><br/>
<sub>163 commits</sub>
</td>
<td align="center">
<a href="https://github.com/genovoxx"><img src="https://github.com/genovoxx.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@genovoxx</b></sub></a><br/>
<sub>108 commits · Frontend</sub>
</td>
<td align="center">
<a href="https://github.com/sumerjohal"><img src="https://github.com/sumerjohal.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@sumerjohal</b></sub></a><br/>
<sub>AgStack core team</sub>
</td>
<td align="center">
<a href="https://github.com/MBJuric"><img src="https://github.com/MBJuric.png" width="70" height="70" style="border-radius:50%"/><br/><sub><b>@MBJuric</b></sub></a><br/>
<sub>Mobile & Blockchain</sub>
</td>
</tr>
</table>

---

## ✨ Key Features (INATrace 2.x)

INATrace 2 is a major release that significantly expanded the platform:

### Value Chain Management
- ✅ **Generic value chain support** — configure different commodity chains with specific settings
- ✅ **Multi-tenant architecture** — multiple organizations in one deployment
- ✅ **Company profiles** — facilities, processing actions, farmers, and collectors per company
- ✅ **Product admin companies** — delegated product management within value chains

### Supply Chain Operations
- ✅ **Processing actions** — multiple types including bulk purchases for semi-products
- ✅ **Final products** — output of final processing, configurable by product admins
- ✅ **Customer orders** — linked to final products
- ✅ **Purchases, processing & payments** — comprehensive workflow management

### Data & Compliance
- ✅ **Farmer import** — bulk import from Excel files
- ✅ **Processing evidence fields** — configurable in system settings
- ✅ **Multi-language support** — translations for facilities, actions, semi-products, and evidence types
- ✅ **Multi-currency** — configurable currencies with daily exchange rate sync via [exchangeratesapi.io](https://exchangeratesapi.io/)
- ✅ **EUDR compliance** — EU Deforestation Regulation support through polygon mapping and traceability

### Mobile & Field
- ✅ **GPS polygon mapping** — field boundary capture on mobile devices
- ✅ **Offline-first** — full data collection without connectivity
- ✅ **Auto-sync** — seamless data synchronization when online
- ✅ **Available on** [Google Play](https://play.google.com/store/apps/details?hl=en-US&id=com.sunesis.inatrace) and [App Store](https://apps.apple.com/us/app/inatrace/id6572305162)

---

## 🚀 Getting Started

### Full Platform Deployment

To deploy the complete INATrace platform, you'll need to set up:

1. **Database** — MySQL 8.4+ (can use Docker)
2. **Backend** — Java 17 + Spring Boot API server
3. **Frontend** — Angular web application
4. **Mobile** *(optional)* — React Native app via Expo
5. **Blockchain** *(optional)* — Hyperledger Fabric network

```bash
# 1. Clone all repositories
git clone https://github.com/agstack/inatrace-backend.git
git clone https://github.com/agstack/inatrace-frontend.git
git clone https://github.com/agstack/inatrace-mobile.git
git clone https://github.com/agstack/inatrace-coffee-network.git

# 2. Start MySQL
docker run --name inatrace-mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=inatrace \
  -e MYSQL_USER=inatrace \
  -e MYSQL_PASSWORD=inatrace \
  -p 3306:3306 -d mysql:8.4.6

# 3. Start Backend (see inatrace-backend README for full config)
cd inatrace-backend
# Configure application.properties
mvn spring-boot:run

# 4. Start Frontend
cd ../inatrace-frontend
npm install && ng serve

# 5. (Optional) Start Mobile
cd ../inatrace-mobile
npm install && npx expo run:android
```

For detailed setup instructions, see each repository's README.

---

## 🔗 Related Projects

INATrace is part of the broader **AgStack Foundation** ecosystem of open digital infrastructure for agriculture:

| Project | Description |
|---------|-------------|
| [TraceFoodChain](https://github.com/agstack/tracefoodchain) | Flutter app for tracing goods along food production chains |
| [TerraTrac](https://github.com/agstack/TerraTrac-field-app) | EUDR compliance mobile app by TechnoServe Labs |
| [Asset Registry](https://github.com/agstack/asset-registry) | Geospatial asset registration for agricultural land |
| [OpenAgri Platform](https://github.com/agstack/OpenAgri-Bootstrap-Deployment) | Modular digital agriculture microservices |

👉 **See all AgStack projects:** [github.com/agstack](https://github.com/agstack)

---

## 📜 License

All INATrace repositories are licensed under the **Mozilla Public License 2.0** (MPL-2.0).

---

## 🤝 Contributing

We welcome contributions! To get started:

1. Browse [open issues](https://github.com/issues?q=is%3Aopen+is%3Aissue+org%3Aagstack+inatrace) across INATrace repos
2. Fork the relevant repository
3. Create a feature branch and submit a pull request
4. See individual repo READMEs for development setup details

---

<div align="center">

**[Backend](https://github.com/agstack/inatrace-backend)** · **[Frontend](https://github.com/agstack/inatrace-frontend)** · **[Mobile](https://github.com/agstack/inatrace-mobile)** · **[Blockchain](https://github.com/agstack/inatrace-coffee-network)**

Part of [The AgStack Foundation](https://github.com/agstack) · A [Linux Foundation](https://www.linuxfoundation.org/) Project

</div>



<div align="center">

  [![.NET 8](https://img.shields.io/badge/.NET_8-%23512bd4?style=for-the-badge&logo=dotnet)](https://dotnet.microsoft.com)
  [![Blazor](https://img.shields.io/badge/Blazor-%23512bd4?style=for-the-badge&logo=blazor)](https://dotnet.microsoft.com/aspnet/blazor)
  [![Bootstrap](https://img.shields.io/badge/Bootstrap-563D7C?style=for-the-badge&logo=bootstrap&logoColor=white)](https://getbootstrap.com)
  [![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=fff)](https://hub.docker.com/)
</div>

# MT-LMS
This is a showcase of a Learning Management System (LMS) I build on Blazor Server for internal use of a small company.

* **Role:** Fullstack Developer
* **Timeline:** September - November 2026 (6 Weeks)
* **Status:** Complete

## [Explore the website!](https://github.com/Pandora1337/mt-lms/wiki/1.-Login)

**Note**:
- The source code for this project is proprietary and confidential. This repository serves as a portfolio case study highlighting the architecture, design, and feature set.

- The website is machine translated to english, as it was built for a non-english speaking company.

**Table of Contents**:
<ol>
  <li><a href="https://github.com/Pandora1337/mt-lms/wiki/1.-Login">Explore the Website!</a></li>
  <li><a href="#features">Features</a></li>
  <li><a href="#tools-and-stack">Tools & Stack</a></li>
  <li>
    <a href="#architecture">Architecture</a>
    <ul>
      <li><a href="#design-requirements">Design Requirements</a></li>
      <li><a href="#summary">Summary</a></li>
      <li><a href="#key-architectural-patterns">Key Architectural Patterns</a></li>
      <li><a href="#documentation">Documentation</a></li>
    </ul>
  </li>
  <li><a href="#hosting">Hosting</a></li>
  <li><a href="#deployment">Deployment</a></li>
</ol>

---

### Features
- **Role-Based Access Control (RBAC)** - Using `ASP.NET Core Identity` with custom user profiles
- **User Roles**:
  - **User** - Has access to view Courses, Tests, and Additional Materials
  - **Manager** - Can create and edit their respective Users, and gets access to Manager-only Courses and Tests.
  - **Admin** - Can create and edit Courses, Tests, and Additional Materials. Can also edit all users and change their roles.
- **Real Time Interactivity** - Shows Thumbnail Previews for Courses, Generates Markdown Preview for lessons editing, and real time quiz taking
- **Automated Tests Grading** - Automatically grades Tests on their completion, and saves the results for Manager review
- **Progress Tracking** - Unseen Lessons are marked for the user, and each Manager sees their respective users' Test progression
- **Course Sorting** - Find the most important, most recent, or any unseen courses!
- **Mobile friendly** - All UI adapts dynamically to a phone or tablet screen

---

### Tools and Stack

| Category | Tool / Platform |
| :--- | :--- |
| **Frontend & Backend** | Blazor Server (.NET 8.0)
| **Styling** | Bootstrap 5
| **Database** | MariaDB
| **Database ORM** | Entity Framework Core (Pomelo.EntityFrameworkCore.MySql)
| **CI/CD** | GitHub Actions, Ansible
| **Deployment** | Docker
| **Reverse Proxy** | Caddy
| **Stress Testing** | Grafana's k6

---

## Architecture
### Design Requirements:

- The goal is to provide informational materials for employee training and costumer request processing for internal use of a small to medium sized business.

- To further assist in training, a testing feature is required to assess onboarding progress.

- Lack of a dedicated IT department means portability, flexibility, and low-maintenance are paramount.

- All materials need to be editable by a non-technical user (no server files editing).

### Summary

Blazor is a full-stack framework for both client-side and server-side unified development in C#, instead of JS. This reduces context-switching, simplifies data handling through logic sharing (Validation, Data Models, Logic), and allows access to NuGet Packages

Blazor Server supports both Server-Side Rendering (SSR) and Interactive Rendering (Over WebSocket connection and SignalR), which provides classic forms and REST APIs, as well as advanced real time interactivity for quizzes/tests, and Markdown rendering with preview.

Since interactivity requires a dedicated stateful circuit on the server for each active client, it is used only when necessary (Tests and Course editing). Furthermore, keeping business logic strictly decoupled from UI rendering components is vital to maintain consistently low hosting profile.

### Key Architectural Patterns

Project Structure
```
.deploy/                     # Ansible Playbooks, Env configs, docker-compose.yml
.vscode/                     # VSCode tasks and launch options
wwwroot/                     # Static web assets and style sheets
Services/                    # Scoped and Global handlers of logic and data
Models/                      # Generic Models and Interfaces
└── <Feature Models>         # Feature-specific Models and DTOs
Data/                        # DB Migrations, App DB Context, Class Extensions
Components/                  # All .razor
├── <Feature Components>/    # Feature-sliced pages and localized components
│   └── <Feature>Endpoint.cs # API endpoint for the feature calls from SSR pages
├── <Shared Components>/     # Reusable generic UI components
├── Account/                 # Authentication wrappers for Identity
├── Layout/                  # MainLayout, NavMenu
├── Pages/                   # Main routable pages
└── App.razor                # Router configuration & cascading security
Program.cs                   # Composition root and DI configuration
```

To avoid massive, unmaintainable razor files, UI elements are broken down into 3 distinct categories:
- Container Components (Pages): Routable components (`@page "/orders"`) that inject application services/mediators, coordinate data fetching, handle lifecycle triggers, and manage overall page state. The "Root" of a feature.

- Presentation Components ("Components"): Reusable visual elements that accept parameters (`[Parameter]`) and throw events (`[Parameter] EventCallback`). They contain no direct data-access code.

- Localised Pages (Routable Components): Scoped pages to a specific feature. Stored in feature directories due to thematic relevance. Are Never the "Root" of a feature.

State is managed in 2 distinct scopes:
1. Component State: Local variables inside a single .razor file for UI-only toggles.

2. Circuit State (Scoped Services): Injected via DI with a Scoped lifetime. Because each user session owns a unique SignalR circuit.

### Documentation

Extensive documentation was written for each service, architectural decisions during implementation, and how they interact with each other.

I also wrote separate guidebooks with step-by-step explanations and commands for seting up a new host, deploying the application stack, migration procedure, and database backup/restoration procedure

## Hosting

**VPS**
Since the website is for internal use in a small company targeting ~200 users, I decided that a VPS hosting both the website and database provides best cost-efficiency and ease of management.

**Environment**
Being environment agnostic was an important consideration: to have the flexibility to migrate elsewhere if required. For that reason Docker was chosen to orchestrate blazor server, database, and reverse-proxy using a docker-compose file.

This makes app install portable and easily setup with a single command, provided the host is properly configured.

**Reverse Proxy**:
Caddy was chosen as a reverse proxy for similar reasons. Despite its newness, it provides several key advantages over NGINX that make sense for low-maintenance service. The biggest being auto-renewal of Let's Encrypt certificates, and easy to read configuration file (the Caddyfile).

Using a separate reverse proxy instead of Microsoft's Kestrel on IIS also aligns best with flexible hosting strategy.

**Backups**
Periodic updates are made automatically using a cronjob script that connects to the local database. They can be also created manually and downloaded using an Ansible playbook.

Off-site backups are sent via SFTP weekly to an archive bucket.

## Deployment

**Setup**
Setup of the target VPS is done using Ansible Playbooks to update and install required packages (such as Docker), setup swap space, firewall, and SSH keys.

**App Deployment**
Deployment is also handled by an Ansible playbook. It copies environment files (docker-compose, Caddyfile, .env), and runs `docker compose up -d`. It also restarts the containers to apply any changes while running.

**CI/CD**
When a new feature branch merges on GitHub, an Action creates and publishes a new app image to Docker Hub, as well as trigger image pull on the host if successful.


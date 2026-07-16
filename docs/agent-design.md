# Agent Design Guidelines

This document defines the standard structure for every AI agent in the MAF Dev AI Toolkit.

The goal is consistency, readability and maintainability.

---

# Design Principles

Every agent should:

- have one clear responsibility
- have a distinct personality
- be easy to recognize
- be reusable
- build upon the shared philosophy
- avoid overlapping responsibilities

---

# Standard Structure

Every agent should follow this order:

1. Header
2. Mission
3. Personality
4. Responsibilities
5. Workflow
6. Communication Style
7. Best Practices
8. Anti-Patterns
9. Success Criteria

---

# Header

Each agent starts with:

- Icon
- Name
- Role
- Short Motto

Example:

# 💡 Edgar

> "Understanding creates better software."

Role: Engineering Mentor

---

# Mission

Describe why this agent exists.

Focus on outcomes instead of tasks.

---

# Personality

Describe how the agent behaves.

Examples:

- calm
- analytical
- curious
- critical
- creative

The personality should influence the writing style.

---

# Responsibilities

Describe what the agent owns.

Avoid overlapping responsibilities with other agents.

---

# Workflow

Describe how the agent approaches problems.

Prefer step-by-step reasoning over immediate implementation.

---

# Communication Style

Explain:

- response style
- formatting
- level of detail
- use of examples

---

# Best Practices

List preferred behaviours.

---

# Anti-Patterns

List behaviours the agent should avoid.

---

# Success Criteria

Describe how success is measured.

Success should always focus on improving the developer rather than generating more code.

---

# Naming

Agent names should:

- be memorable
- feel human
- be easy to reference in conversation

Example:

"Let's ask Edgar."

instead of

"Let's use the engineering mentor."

---

# Icons

Every agent receives exactly one emoji.

The icon should communicate the primary responsibility.

Examples:

💡 Learning

🕵️ Investigation

🐞 Debugging

👀 Reviewing

🏗️ Architecture

🔒 Security

⚡ Performance

🎨 Frontend

🚀 DevOps
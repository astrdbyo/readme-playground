# CI / CD Workflow Guide

This document explains **why these workflows exist**, **what each one does**, and **how to use them**.  
It is written for future reference, so you do not need to re-learn the entire CI/CD concept again.

---

## Purpose

This repository uses **GitHub Actions** to automate the full software delivery lifecycle:

1. **Validate code before merge**
2. **Build Docker images**
3. **Deploy safely to staging**
4. **Deploy explicitly to production**
5. **Create a production release record**

Each step has **one responsibility only**.  
Nothing is automatic in production by design.

---

## Workflow Overview


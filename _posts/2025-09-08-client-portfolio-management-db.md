---
title: "Client Portfolio Management Database"
date: 2025-09-08
media_subpath: /assets/posts/2025-09-08-client-portfolio-management-db
published: true
layout: post
categories:
- databases
tags:
- database
- sql
- backend
- python
- postgresql
---

## Repository

For running this project and documentation, visit the GitHub repo: [client-portfolio-management-db](https://github.com/jbrowne7/client-portfolio-risk-management-DB?tab=readme-ov-file#overview)


## Overview

Built a system using postgreSQL, python, and SQL to store data about an investment company's clients, their portfolios, assets, asset prices, and trades on clients portfolios.

## Motivations

I built this project to improve my skills mainly with SQL and my knowledge of relational databases.I wanted to practice designing database schemas and implementing different kinds of queries for pulling information. 

## What I Built

A comprehensive database system for managing client portfolios with:

- **Relational Database Schema** - PostgreSQL database with tables for clients, portfolios, assets, trades, and price history
- **Python Backend** - Database interaction layer using psycopg2 for CRUD operations
- **CLI Application** - Command-line interface for managing all database operations
- **Multi-Asset Support** - Handles stocks, bonds, forex, and cryptocurrency across different currencies
- **Trade Management System** - Buy/sell transaction recording
- **Database Migration Workflow** - Scripts for database schema updates


## Technologies Used

- **PostgreSQL** - Primary database for storing client and portfolio data
- **Python** - Backend development and database interaction
- **SQL** - Complex queries for data retrieval and analysis
- **psycopg2** - PostgreSQL adapter for Python
- **Database Design** - Normalized relational schema design

## Key Features

- **Client Management** - Store and manage client information with first/last names
- **Portfolio Tracking** - Track multiple portfolios per client with cash balances
- **Asset Management** - Support for stocks, bonds, forex, and crypto across multiple currencies
- **Trade Recording** - Complete trade history with buy/sell transactions
- **Price Tracking** - Historical price data for assets
- **Data Migration** - Migration workflow for database schema changes
- **CLI Interface** - Command-line interface for all operations

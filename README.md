# 🏦 Bureau de Change Management System

> A relational database system for managing foreign exchange operations, built on real compliance requirements from a live financial services environment.

---

## 📌 Overview

Designed and implemented a fully operational SQLite database system to manage foreign exchange transactions at a Bureau de Change. The system enforces AML compliance, real-time liquidity checks, KYC verification, role-based access control, and an immutable audit trail — replacing error-prone manual spreadsheets with a structured, auditable data infrastructure.

This project was built from direct operational experience managing $500K+ in monthly FX transactions as a Data & Operations Analyst.

**Tools:** SQL, SQLite, DB Browser for SQLite

---

## 🗄️ Database Schema

The system consists of 10 interrelated tables:

| Table | Description |
|---|---|
| `Customer` | Customer records with KYC status tracking |
| `Employee` | Staff records with role assignments and active status |
| `Role` | Permission levels: Teller, Supervisor, Compliance Officer |
| `Currency` | Supported currencies: USD, EUR, GBP, ZAR, ZWL, BWP |
| `ExchangeRates` | Official buy/sell rates with timestamp history |
| `Transactions` | Core FX transaction records (51 entries) |
| `CashInventory` | Real-time float balances per currency |
| `ComplianceFlag` | AML flags with reasons and active status |
| `KYCAuditLog` | KYC verification history per customer (26 entries) |
| `AuditLog` | Immutable system-wide activity log (60 entries) |

---

## ⚙️ Core Business Rules Enforced

**Rate Control** — Exchange rates are system-calculated using the official buy rate. Tellers cannot override computed forex payouts.

**Liquidity Management** — Before every transaction, the system checks `CashInventory` for sufficient float. Transactions are blocked if the teller is illiquid for the currency to be disbursed. Balances update automatically after each successful trade.

**KYC Enforcement** — Customer KYC details are mandatory for onboarding. Transactions exceeding the regulatory threshold (equivalent of USD $500, per RBZ guidelines) are held if KYC is incomplete or unverified.

**Role-Based Access** — Only Employees with `Manager` role can update daily exchange rates. Inactive employees cannot process transactions.

**Immutable Audit Trail** — Every transaction, rate update, and user action is logged in `AuditLog` with an `EmployeeID` and `DateTime` timestamp. Records cannot be altered.

**Single Active Rate** — At any given time, only one `ExchangeRate` record per `CurrencyCode` can be active, preventing rate conflicts.

---

## 🚨 AML Compliance Features

The `ComplianceFlag` table captures suspicious activity with structured flag reasons including:

- Unusual currency pair patterns
- Customers on internal watchlists
- Multiple transactions just below the regulatory reporting threshold (structuring)

---

## 🗂️ Repository Structure

```
bdc-management-system/
│
├── BDC_Management_system.db       # SQLite database with full schema and sample data
├── BDC_Management_system.sqbpro   # DB Browser project file
├── report.pdf                     # System design report with ERD and business rules
└── README.md                      # Project overview
```

---

## 💡 Sample Queries

```sql
-- Check KYC status before processing a high-value transaction
SELECT t.TransactionID, c.FirstName, c.LastName, c.KYCStatus, t.AmountReceived,
    CASE
        WHEN t.AmountReceived > 500 AND c.KYCStatus != 'Verified'
        THEN 'Hold'
        ELSE 'Proceed'
    END AS TransactionAction
FROM Transactions t
JOIN Customer c ON t.FK_CustomerID = c.CustomerID;

-- View all active compliance flags with transaction details
SELECT cf.FlagID, cf.FlagReason, cf.FlagDate, t.AmountReceived, t.FK_CurrencyReceived
FROM ComplianceFlag cf
JOIN Transactions t ON cf.FK_TransactionID = t.TransactionID
WHERE cf.Status = 'Active';

-- Audit trail for a specific employee
SELECT ActionType, TableAffected, RecordID, DateTime
FROM AuditLog
WHERE FK_EmployeeID = 5
ORDER BY DateTime DESC;
```

---

## 👤 Author

**Akheem Amisi**  
MPS Analytics — Northeastern University  
[LinkedIn](https://linkedin.com/in/akheem-amisi)
